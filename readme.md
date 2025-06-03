ait ky.post('httpirst. When called like that, an appropriate `Accept` header will be set depending on the body method used. Unlike the `Body` methods of `window.Fetch`, these will throw an `HTTPError` if the response status is not in the range of `200...299`. Also, `.jsScript:** Accepts an optional type parameter for use with JSON responses (see [`ky()`](#kyinput-options)).

#### input

Type: `string` | `URL` | `Request`

Same as [`fetch` input](https://developer.mozilla.org/en-US/docs/Web/API/Requema [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) instance as `input`, any URL altering options (such as `prefixUrl`) will be ignored.

#### options

Type: `object`

Same as [`fetch` options](https://developer.mozilla.org/en-US/docs/Web/API/fetch#options), plus the following additional options:

##### method

Type: `string`\
Default: `'get'`

HTTP method used to make the request.

Internally, the standard methods (`GET`, `POST`, `PUT`, `PATCH`, `HEAD` and `DELETE`) are uppercased in order to avoid server errors due to case sensitivity.

##### json

Type: `object` and any other value accepted by [`JSON.stringify()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

Shortcut for sending JSON. Use this instead of the `body` option. Accepts any plain object or value, which will be `JSON.stringify()`'d and sent in the body with the correct header set.

##### searchParams

Type: `string | object<string, string | number | boolean> | Array<Array<string | number | boolean>> | URLSearchParams`\
Default: `''`

Search parameters to include in the request URL. Setting this will override all existing search parameters in the input URL.

Accepts any value supported by [`URLSearchParams()`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/URLSearchParams).

##### prefixUrl

Type: `string | URL`

A prefix to prepend to the `input` URL when making the request. It can be any valid URL, either relative or absolute. A trailing slash `/` is optional and will be added automatically, if needed, when it is joined with `input`. Only takes effect when `input` is a string. The `input` argument cannot start with a slash `/` when using this option.

Useful when used with [`ky.extend()`](#kyextenddefaultoptions) to create niche-specific Ky-instances.

```js
import ky from 'ky';

// On https://example.com

const response = await ky('unicorn', {prefixUrl: '/api'});
//=> 'https://example.com/api/unicorn'

const response2 = await ky('unicorn', {prefixUrl: 'https://cats.com'});
//=> 'https://cats.com/unicorn'
```

Notes:
 - After `prefixUrl` and `input` are joined, the result is resolved against the [base URL](https://developer.mozilla.org/en-US/docs/Web/API/Node/baseURI) of the page (if any).
 - Leading slashes in `input` are disallowed when using this option to enforce consistency and avoid confusion about how the `input` URL is handled, given that `input` will not follow the normal URL resolution rules when `prefixUrl` is being used, which changes the meaning of a leading slash.

##### retry

Type: `object | number`\
Default:
- `limit`: `2`
- `methods`: `get` `put` `head` `delete` `options` `trace`
- `statusCodes`: [`408`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408) [`413`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/413) [`429`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) [`500`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500) [`502`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/502) [`503`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/503) [`504`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/504)
- `afterStatusCodes`: [`413`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/413), [`429`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429), [`503`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/503)
- `maxRetryAfter`: `undefined`
- `backoffLimit`: `undefined`
- `delay`: `attemptCount => 0.3 * (2 ** (attemptCount - 1)) * 1000`

An object representing `limit`, `methods`, `statusCodes`, `afterStatusCodes`, and `maxRetryAfter` fields for maximum retry count, allowed methods, allowed status codes, status codes allowed to use the [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) time, and maximum [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) time.

If `retry` is a number, it will be used as `limit` and other defaults will remain in place.

If the response provides an HTTP status contained in `afterStatusCodes`, Ky will wait until the date, timeout, or timestamp given in the [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) header has passed to retry the request. If `Retry-After` is missing, the non-standard [`RateLimit-Reset`](https://www.ietf.org/archive/id/draft-polli-ratelimit-headers-05.html#section-3.3) header is used in its place as a fallback. If the provided status code is not in the list, the [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) header will be ignored.

If `maxRetryAfter` is set to `undefined`, it will use `options.timeout`. If [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) header is greater than `maxRetryAfter`, it will use `maxRetryAfter`.

The `backoffLimit` option is the upper limit of the delay per retry in milliseconds.
To clamp the delay, set `backoffLimit` to 1000, for example.
By default, the delay is calculated with `0.3 * (2 ** (attemptCount - 1)) * 1000`. The delay increases exponentially.

The `delay` option can be used to change how the delay between retries is calculated. The function receives one parameter, the attempt count, starting at `1`.

Retries are not triggered following a [timeout](#timeout).

```js
import ky from 'ky';

const json = await ky('https://example.com', {
	retry: {
		limit: 10,
		methods: ['get'],
		statusCodes: [413],
		backoffLimit: 3000
	}
}).json();
```

##### timeout

Type: `number | false`\
Default: `10000`

Timeout in milliseconds for getting a response, including any retries. Can not be greater than 2147483647.
If set to `false`, there will be no timeout.

##### hooks

Type: `object<string, Function[]>`\
Default: `{beforeRequest: [], beforeRetry: [], afterResponse: []}`

Hooks allow modifications during the request lifecycle. Hook functions may be async and are run serially.

###### hooks.beforeRequest

Type: `Function[]`\
Default: `[]`

This hook enables you to modify the request right before it is sent. Ky will make no further changes to the request after this. The hook function receives `request` and `options` as arguments. You could, for example, modify the `request.headers` here.

The hook can return a [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) to replace the outgoing request, or return a [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) to completely avoid making an HTTP request. This can be used to mock a request, check an internal cache, etc. An **important** consideration when returning a request or response from this hook is that any remaining `beforeRequest` hooks will be skipped, so you may want to only return them from the last hook.

```js
import ky from 'ky';

const api = ky.extend({
	hooks: {
		beforeRequest: [
			request => {
				request.headers.set('X-Requested-With', 'ky');
			}
		]
	}
});

const response = await api.get('https://example.com/api/users');
```

###### hooks.beforeRetry

Type: `Function[]`\
Default: `[]`

This hook enables you to modify the request right before retry. Ky will make no further changes to the request after this. The hook function receives an object with the normalized request and options, an error instance, and the retry count. You could, for example, modify `request.headers` here.

If the request received a response, the error will be of type `HTTPError` and the `Response` object will be available at `error.response`. Be aware that some types of errors, such as network errors, inherently mean that a response was not received. In that case, the error will not be an instance of `HTTPError`.

You can prevent Ky from retrying the request by throwing an error. Ky will not handle it in any way and the error will be propagated to the request initiator. The rest of the `beforeRetry` hooks will not be called in this case. Alternatively, you can return the [`ky.stop`](#kystop) symbol to do the same thing but without propagating an error (this has some limitations, see `ky.stop` docs for details).

```js
import ky from 'ky';

const response = await ky('https://example.com', {
	hooks: {
		beforeRetry: [
			async ({request, options, error, retryCount}) => {
				const token = await ky('https://example.com/refresh-token');
				request.headers.set('Authorization', `token ${token}`);
			}
		]
	}
});
```

###### hooks.beforeError

Type: `Function[]`\
Default: `[]`

This hook enables you to modify the `HTTPError` right before it is thrown. The hook function receives a `HTTPError` as an argument and should return an instance of `HTTPError`.

```js
import ky from 'ky';

await ky('https://example.com', {
	hooks: {
		beforeError: [
			error => {
				const {response} = error;
				if (response && response.body) {
					error.name = 'GitHubError';
					error.message = `${response.body.message} (${response.status})`;
				}

				return error;
			}
		]
	}
});
```

###### hooks.afterResponse

Type: `Function[]`\
Default: `[]`

This hook enables you to read and optionally modify the response. The hook function receives normalized request, options, and a clone of the response as arguments. The return value of the hook function will be used by Ky as the response object if it's an instance of [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response).

```js
import ky from 'ky';

const response = await ky('https://example.com', {
	hooks: {
		afterResponse: [
			(_request, _options, response) => {
				// You could do something with the response, for example, logging.
				log(response);

				// Or return a `Response` instance to overwrite the response.
				return new Response('A different response', {status: 200});
			},

			// Or retry with a fresh token on a 403 error
			async (request, options, response) => {
				if (response.status === 403) {
					// Get a fresh token
					const token = await ky('https://example.com/token').text();

					// Retry with the token
					request.headers.set('Authorization', `token ${token}`);

					return ky(request);
				}
			}
		]
	}
});
```

##### throwHttpErrors

Type: `boolean`\
Default: `true`

Throw an `HTTPError` when, after following redirects, the response has a non-2xx status code. To also throw for redirects instead of following them, set the [`redirect`](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters) option to `'manual'`.

Setting this to `false` may be useful if you are checking for resource availability and are expecting error responses.

Note: If `false`, error responses are considered successful and the request will not be retried.

##### onDownloadProgress

Type: `Function`

Download progress event handler.

The function receives these arguments:
- `progress` is an object with the these properties:
- - `percent` is a number between 0 and 1 representing the progress percentage.
- - `transferredBytes` is the number of bytes transferred so far.
- - `totalBytes` is the total number of bytes to be transferred. This is an estimate and may be 0 if the total size cannot be determined.
- `chunk` is an instance of `Uint8Array` containing the data that was sent. Note: It's empty for the first call.

```js
import ky from 'ky';

const response = await ky('https://example.com', {
	onDownloadProgress: (progress, chunk) => {
		// Example output:
		// `0% - 0 of 1271 bytes`
		// `100% - 1271 of 1271 bytes`
		console.log(`${progress.percent * 100}% - ${progress.transferredBytes} of ${progress.totalBytes} bytes`);
	}
});
```

##### onUploadProgress

Type: `Function`

Upload progress event handler.

The function receives these arguments:
- `progress` is an object with the these properties:
- - `percent` is a number between 0 and 1 representing the progress percentage.
- - `transferredBytes` is the number of bytes transferred so far.
- - `totalBytes` is the total number of bytes to be transferred. This is an estimate and may be 0 if the total size cannot be determined.
- `chunk` is an instance of `Uint8Array` containing the data that was sent. Note: It's empty for the last call.

```js
import ky from 'ky';

const response = await ky.post('https://example.com/upload', {
	body: largeFile,
	onUploadProgress: (progress, chunk) => {
		// Example output:
		// `0% - 0 of 1271 bytes`
		// `100% - 1271 of 1271 bytes`
		console.log(`${progress.percent * 100}% - ${progress.transferredBytes} of ${progress.totalBytes} bytes`);
	}
});
```

##### parseJson

Type: `Function`\
Default: `JSON.parse()`

User-defined JSON-parsing function.

Use-cases:
1. Parse JSON via the [`bourne` package](https://github.com/hapijs/bourne) to protect from prototype pollution.
2. Parse JSON with [`reviver` option of `JSON.parse()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse).

```js
import ky from 'ky';
import bourne from '@hapijs/bourne';

const json = await ky('https://example.com', {
	parseJson: text => bourne(text)
}).json();
```

##### stringifyJson

Type: `Function`\
Default: `JSON.stringify()`

User-defined JSON-stringifying function.

Use-cases:
1. Stringify JSON with a custom `replacer` https://github.com/axios/axios)?

See my answer [here](https://twitter.com/sindresorhus/status/1037763588826398720).

#### How is it different from [`r2`](https://github.com/mikeal/r2)?

See my answer in [#10](https://github.com/sindresorhus/ky/issna