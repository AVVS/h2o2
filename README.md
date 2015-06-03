#h2o2

Proxy handler plugin for hapi.js.

[![NPM](https://nodei.co/npm/h2o2.png?downloads=true&stars=true)](https://nodei.co/npm/h2o2/)

[![Build Status](https://secure.travis-ci.org/hapijs/h2o2.png)](http://travis-ci.org/hapijs/h2o2)

Lead Maintainer - [Eran Hammer](https://github.com/hueniverse)

## Introduction

**h2o2** is the proxy handler created for internal use of hapi.

## Options

The proxy handler object has the following properties:

* `host` - upstream service host to proxy requests to. It will have the same path as the cliet request.
* `port` - upstream service port.
* `protocol` - protocol to use when making the request to the proxied host:
    * 'http'
    * 'https'
* `uri` - absolute URI used insted of host, port, protocol, path, and query. Cannot be used with `host`, `port`, `protocol`, or `mapUri`.
* `passThrough` - if set to `true`, it forwards the headers from the client to the upstream service, headers sent from the upstream service will also be forwarded to the client. Defaults to `false`.
* `localStatePassThrough` - if set to`false`, any locally defined state is removed from incoming requests before being sent to the upstream service. This value can be overridden on a per state basis via the `server.state()``passThrough` option. Defaults to `false`
* `acceptEncoding` - if set to `false`, does not pass-through the 'Accept-Encoding' HTTP header which is useful for the `onResponse` post-processing to avoid receiving an encoded response. Can only be used together with `passThrough`. Defaults to `true` (passing header).
* `rejectUnauthorized` - sets the `rejectUnauthorized` property on the https [agent](http://nodejs.org/api/https.html#https_https_request_options_callback) making the request. This value is only used when the proxied server uses TLS/SSL. If set it will override the node.js `rejectUnauthorized` property. If `false` then ssl errors will be ignored. When `true` the server certificate is verified and an 500 response will be sent when verification fails. This shouldn't be used alongside the `agent` setting as the `agent` will be used instead. Defaults to the https agent default value of `true`.
* `xforward` - if set to `true`, sets the 'X-Forwarded-For', 'X-Forwarded-Port', 'X-Forwarded-Proto' headers when making a request to the proxied upstream endpoint. Defaults to `false`.
* `redirects` - the maximum number of HTTP redirections allowed to be followed automatically by the handler. Set to `false` or `0` to disable all redirections (the response will contain the redirection received from the upstream service). If redirections are enabled, no redirections (301, 302, 307, 308) will be passed along to the client, and reaching the maximum allowed redirections will return an error response. Defaults to `false`.
* `timeout` - number of milliseconds before aborting the upstream request. Defaults to `180000` (3 minutes).
* `mapUri` - a function used to map the request URI to the proxied URI. Cannot be used together with `host`, `port`, `protocol`, or `uri`. The function signature is `function(request, callback)` where:
    * `request` - is the incoming [request object](http://hapijs.com/api#request-object).
    * `callback` - is `function(err, uri, headers)` where:
        * `err` - internal error condition.
        * `uri` - the absolute proxy URI.
        * `headers` - optional object where each key is an HTTP request header and the value is the header content.
* `onResponse` - a custom function for processing the response from the upstream service before sending to the client. Useful for custom error handling of responses from the proxied endpoint or other payload manipulation. Function signature is `function(err, res, request, reply, settings, ttl)` where:
    * `err` - internal or upstream error returned from attempting to contact the upstream proxy.
    * `res` - the node response object received from the upstream service. `res` is a readable stream (use the [wreck](https://github.com/hapijs/wreck) module `read` method to easily convert it to a Buffer or string).
    * `request` - is the incoming [request object](http://hapijs.com/api#request-object).
    * `reply` - the [reply interface](http://hapijs.com/api#reply-interface) function.
    * `settings` - the proxy handler configuration.
    * `ttl` - the upstream TTL in milliseconds if `proxy.ttl` it set to `'upstream'` and the upstream response included a valid 'Cache-Control' header with 'max-age'.
* `ttl` - if set to `'upstream'`, applies the upstream response caching policy to the response using the `response.ttl()` method (or passed as an argument to the `onResponse` method if provided).
* `agent` - a node [http(s) agent](http://nodejs.org/api/http.html#http_class_http_agent) to be used for connections to upstream server.
* `maxSockets` - sets the maximum number of sockets available per outgoing proxy host connection. `false` means use the **wreck** module default value (Infinity). Does not affect non-proxy outgoing client connections. Defaults to Infinity.

## Usage

As one of the built-in handlers of hapi, it is used through the route configuration object.

```javascript
server.route({
    method: 'GET'
    path: '/',
    handler: {
        proxy: {
            host: '10.33.33.1',
            port: '80'
            protocol: 'https'
        }
    }
});
```
