# Supported tags and respective `Dockerfile` links

-	[`4.1`, `latest` (*Dockerfile*)](https://github.com/globalgiving/docker-varnish-logging/blob/master/Dockerfile)
-	[`4.0` (*Dockerfile*)](https://github.com/globalgiving/docker-varnish-logging/blob/4.0/Dockerfile)

# Varnish Docker container with logging

> Debian 8 - Jessie
> Varnish 4.1.3
> varnishncsa logging to stdout

## Usage

To use this container, you will need to provide your custom config.vcl (which is usually the case).

```
docker run -d \
  --link web-app:backend-host \
  --volumes-from web-app \
  --env 'VCL_CONFIG=/data/path/to/varnish.vcl' \
  --log-driver=syslog --log-opt syslog-address=tcp://192.168.0.42:123 \
  globalgiving/varnish
```

In the above example we assume that:
* You have your application running inside `web-app` container and web server there is running on port 80 (although you don't need to expose that port, as we use --link and varnish will connect directly to it)
* `web-app` container has `/data` volume with `varnish.vcl` somewhere there
* `web-app` is aliased inside varnish container as `backend-host`
* `192.168.0.42` is running a daemon on port `123` which can consume syslog data.
* Your `varnish.vcl` should contain at least backend definition like this:  
```
backend default {
    .host = "backend-host";
    .port = "80";
}
```

## Environmental variables

You can configure Varnish daemon by following env variables:

> **VCL_CONFIG** `/etc/varnish/default.vcl`  
> **CACHE_SIZE** `64m`  
> **VARNISHD_PARAMS** `-p default_ttl=3600 -p default_grace=3600`

## Logging Output

The output of varnishncsa is piped to the standard out of the container, allowing the docker daemon to read it and direct that data to anywhere you wish.

The format ([variable definitions](https://www.varnish-cache.org/docs/4.1/reference/varnishncsa.html)) passed to varnishncsa:
```
[%{x-forwarded-for}i] %u %t "%r" %s %b %T "%{Referer}i" "%{User-agent}i" %{Varnish:handling}x
```

## Acknowledgements

+ Based heavily on [million12/varnish](https://hub.docker.com/r/million12/varnish/). Thank you!

