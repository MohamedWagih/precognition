# Laravel Precognition

<a href="https://github.com/laravel/precognition/actions"><img src="https://github.com/laravel/precognition/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://www.npmjs.com/package/laravel-precognition"><img src="https://img.shields.io/npm/dt/laravel-precognition" alt="Total Downloads"></a>
<a href="https://www.npmjs.com/package/laravel-precognition"><img src="https://img.shields.io/npm/v/laravel-precognition" alt="Latest Stable Version"></a>
<a href="https://www.npmjs.com/package/laravel-precognition"><img src="https://img.shields.io/npm/l/laravel-precognition" alt="License"></a>

## Introduction

This library provides a thin wrapper around the Axios client to help make Precognition requests. To get started you should install the package:

```sh
npm install laravel-precognition
```

Every request sent via the helper will be a Precognition request. The available request methods, which all return a `Promise`, are:

```js
import precog from 'laravel-precognition';

precog.get(url, config);
precog.post(url, data, config);
precog.patch(url, data, config);
precog.put(url, data, config);
precog.delete(url, config);
```

The `config` parameter is the [Axios' configuration](https://axios-http.com/docs/req_config) with some additional Precognition options as outlined below.

### Handling Successful Responses

A `204 No Content` response with an included `Precognition: true` header indicates that a Precognition request was successful. The `onPrecognitionSuccess` option can be used to handle these responses:

```js
precog.post(url, data, {
    onPrecognitionSuccess: (response) => {
        // Precognition was successful...
    },
});
```

The function receives the [Axios response](https://axios-http.com/docs/res_schema).

### Handling Validation Responses

As validation is a common use-case for Precognition, we have included validation specific affordances. To handle a Laravel validation error response, you may use the `onValidationError` option:

```js
precog.post(url, data, {
    onValidationError: (errors, axiosError) => {
        usernameError = errors.username?[0];
    },
});
```

The function receives the `errors` object from the [validation response](https://laravel.com/docs/validation#validation-error-response-format) and the [Axios error](https://axios-http.com/docs/handling_errors).

### Specifying Inputs For Validation

One of the features of Precognition is the ability to specify the inputs that you would like to run validation rules against. To use this feature you should pass a list of input names to the `validate` option:

```js
precog.post('/users', { ... }, {
    validate: ['username', 'email'],
    onValidationError: (errors, axiosError) => {
        // ...
    },
});
```

### Handling Error Responses

There are a few common error responses that Precognition requests may return. The following outline some options to handle those responses:

```js
precog.post(url, data, {
    onUnauthorized: (response, axiosError) => /* ... */,
    onForbidden: (response, axiosError) => /* ... */,
    onNotFound: (response, axiosError) => /* ... */,
    onConflict: (response, axiosError) => /* ... */,
    onLocked: (response, axiosError) => /* ... */,
});
```

These functions receive the [Axios response](https://axios-http.com/docs/res_schema) and the [Axios error](https://axios-http.com/docs/handling_errors).

### Handling Other Responses

You may also handle the above types and additional response types as you normally would via `.then()`, `.catch()`, and `.finally()`.

```js
loading = true;

precog.post(url, data, { /* ... */ })
       .catch((error) => {
           if (error.response?.status === 418) {
               // ...
           }
       })
       .finally(() => loading = false);
```

### Receiving Non-Precognition Responses

If the Precognition client receives a server response that does not have the `Precognition: true` header set, it will throw an error. You should ensure that the Precognition middleware is in place for the route.

### Automatically Aborting Stale Request

When an [`AbortController` or `CancelToken`](https://axios-http.com/docs/cancellation) are not passed in the request configuration, the Precognition client automatically aborts any still in-flight requests when a new request is made, but only if the new request matches a previous request's signature. It identifies requests by combining the request method and URL.

In the following example, because the method and URL match for both requests, if request 1 is still waiting on a response when request 2 is fired, request 1 will be aborted.

```js
// Request 1
precog.post('/projects/5', { name: 'Laravel' })

// Request 2
precog.post('/projects/5', { name: 'Laravel', repo: 'laravel/framework' })
```

If the URL or the method do not match, then the request would not be aborted:

```js
precog.post('/projects/5', { name: 'Laravel' })

precog.post('/repositories/5', { name: 'Laravel' })
```

You may customize how the Precognition client identifies requests by passing a callback to `useRequestIdResolver`:

```js
import precog from 'laravel-precognition';

precog.useRequestIdResolver((config, axios) => config.headers['Request-Id'])
```

It is also possible to specify the unique identifier on a per request basis:

```js
precog.post('/projects/5', form.data(), {
    requestId: 'unique-id-1',
})

precog.post('/projects/5', form.data(), {
    requestId: 'unique-id-2',
})
```

If you would like to disable this feature globally, you should return `null` from the callback passed to `useRequestIdResolver`:

```js
precog.useRequestIdResolver(() => null)
```

You can also disable to feature on a per request basis by passing `null` as the `requestId` option:

```js
precog.post('/projects/5', form.data(), {
    requestId: null,
})
```

### Using An Existing Axios Instance

If your application already configures an Axios instance, you may instruct Precognition to use that instance by calling `precognition.use(axios)`:

```js
import axios from 'axios';
import precognition from 'laravel-precognition';

// Configure Axios...

window.axios = axios;
axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

// Use the configured instance...

window.precog = precognition.use(axios)
```

## Contributing

Thank you for considering contributing to Laravel Precognition! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Code of Conduct

In order to ensure that the Laravel community is welcoming to all, please review and abide by the [Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).

## Security Vulnerabilities

Please review [our security policy](https://github.com/laravel/precognition/security/policy) on how to report security vulnerabilities.

## License

Laravel Precognition is open-sourced software licensed under the [MIT license](LICENSE.md).
