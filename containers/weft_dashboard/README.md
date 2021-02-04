# Weft Dashboard

This is a container providing a Dashboard, running on Nginx.

## License (This Directory Only)

The demo dashboard [site/](site/) is (Apache 2 licensed) https://github.com/hivemq/hivemq-mqtt-web-client, which will eventually be torn out entirely.

## External Services

* HTTPS: Port 443/tcp

## Build (Using Docker BuildX)

These instructions let you build with a cross-compile (i.e., build on X86 for RasPi).

-- If you haven't set up a BuildX builder yet:

```
docker buildx create --name nubuilder --use
docker buildx inspect --bootstrap
```

-- To build this image and push it to the registry:

```
THE_REGISTRY=ghcr.io/ussjoin
THE_IMAGE=weft_dashboard
docker buildx build . -t #THE_REGISTRY/$THE_IMAGE --platform linux/arm/v7 --load
docker push $THE_REGISTRY/$THE_IMAGE
```

## Build (Native)

```
THE_REGISTRY=ghcr.io/ussjoin
THE_IMAGE=weft_dashboard
docker build -t $THE_REGISTRY/$THE_IMAGE .
docker push $THE_REGISTRY/$THE_IMAGE
```


## Run

```
THE_REGISTRY=ghcr.io/ussjoin
docker run -d --rm \
    -p 8000:80 \
    $THE_REGISTRY/weft_dashboard
```

If you want to check on Nginx while it runs:
`docker exec -ti <containerid> /bin/sh`

