# Weft Time

This is a container providing GPS-linked time services.

## Weft Network Object

```
{
  "type": "time",
  "time": 1598222927,
  "coordinates": {
    "lat": 48.5717611,
    "lon": -123.1611707
  }
}
```

This is generated and sent in [config/send_loc](config/send_loc).

## External Services

* NTP: Port 123/udp
* GPSD: Port 2947/tcp

Both are happy to respond to queries, so if you're running this container as part of a group, this makes an excellent time and location server.

## Build (Using Docker BuildX)

These instructions let you build with a cross-compile (i.e., build on X86 for RasPi).

-- If you haven't set up a BuildX builder yet:

```
docker buildx create --name nubuilder --use
docker buildx inspect --bootstrap
```

-- To build this image and push it to theregistry:

```
THE_REGISTRY=fun.horse
THE_IMAGE=weft_time
docker buildx build . -t $THE_REGISTRY/$THE_IMAGE --platform linux/arm/v7 --load
docker push $THE_REGISTRY/$THE_IMAGE
```

## Build (Native)

THE_REGISTRY=fun.horse
THE_IMAGE=weft_time
docker build -t $THE_REGISTRY/$THE_IMAGE .
docker push $THE_REGISTRY/$THE_IMAGE


## Config

Look in the `config` directory. These configs are placed in during build, not at runtime. Supervisord keeps chrony running.

## Run

```
THE_REGISTRY=fun.horse
docker run -d --rm --cap-add SYS_TIME \
    -p 123:123/udp \
    -p 2947:2947/tcp \
    -e MQTT_HOST=foo \
    -e MQTT_PORT=foo \
    --device=/dev/ttyACM0 \
    $THE_REGISTRY/weft_time
```


If you want to check on Chrony while it runs:
`docker exec -ti <containerid> /bin/sh`

