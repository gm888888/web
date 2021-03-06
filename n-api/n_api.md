
<!--introduced_in=v8.0.0-->
<!-- type=misc -->

> Stability: 2 - Stable

N-API (pronounced N as in the letter, followed by API)
is an API for building native Addons. It is independent from
the underlying JavaScript runtime (for example, V8) and is maintained as part of
Node.js itself. This API will be Application Binary Interface (ABI) stable
across versions of Node.js. It is intended to insulate Addons from
changes in the underlying JavaScript engine and allow modules
compiled for one major version to run on later major versions of Node.js without
recompilation. The [ABI Stability][] guide provides a more in-depth explanation.

Addons are built/packaged with the same approach/tools outlined in the section
titled [C++ Addons][]. The only difference is the set of APIs that are used by
the native code. Instead of using the V8 or [Native Abstractions for Node.js][]
APIs, the functions available in the N-API are used.

APIs exposed by N-API are generally used to create and manipulate
JavaScript values. Concepts and operations generally map to ideas specified
in the ECMA-262 Language Specification. The APIs have the following
properties:

* All N-API calls return a status code of type `napi_status`. This
  status indicates whether the API call succeeded or failed.
* The API's return value is passed via an out parameter.
* All JavaScript values are abstracted behind an opaque type named
  `napi_value`.
* In case of an error status code, additional information can be obtained
  using `napi_get_last_error_info`. More information can be found in the error
  handling section [Error handling][].

The N-API is a C API that ensures ABI stability across Node.js versions
and different compiler levels. A C++ API can be easier to use.
To support using C++, the project maintains a
C++ wrapper module called [`node-addon-api`][].
This wrapper provides an inlineable C++ API. Binaries built
with `node-addon-api` will depend on the symbols for the N-API C-based
functions exported by Node.js. `node-addon-api` is a more
efficient way to write code that calls N-API. Take, for example, the
following `node-addon-api` code. The first section shows the
`node-addon-api` code and the second section shows what actually gets
used in the addon.

```cpp
Object obj = Object::New(env);
obj["foo"] = String::New(env, "bar");
```

```cpp
napi_status status;
napi_value object, string;
status = napi_create_object(env, &object);
if (status != napi_ok) {
  napi_throw_error(env, ...);
  return;
}

status = napi_create_string_utf8(env, "bar", NAPI_AUTO_LENGTH, &string);
if (status != napi_ok) {
  napi_throw_error(env, ...);
  return;
}

status = napi_set_named_property(env, object, "foo", string);
if (status != napi_ok) {
  napi_throw_error(env, ...);
  return;
}
```

The end result is that the addon only uses the exported C APIs. As a result,
it still gets the benefits of the ABI stability provided by the C API.

When using `node-addon-api` instead of the C APIs, start with the API [docs][]
for `node-addon-api`.

The [N-API Resource](https://nodejs.github.io/node-addon-examples/)??offers an
excellent orientation and tips for developers just getting started with N-API
and `node-addon-api`.

