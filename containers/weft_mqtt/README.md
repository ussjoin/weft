# Weft MQTT

This is a container providing an MQTT broker (Mosquitto). (I can't use the official Mosquitto docker image because it's not in the right arch.)

## License (This Directory Only)

Much of the code in this folder is copied or derived from https://github.com/eclipse/mosquitto. 
The license: Eclipse Public/Distribution Licenses. https://github.com/eclipse/mosquitto/blob/master/LICENSE.txt

All other information in this directory is under the MIT license.

## External Services

* MQTT: Port 1883/tcp
* MQTT over Websockets: Port 9001/tcp


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
THE_IMAGE=weft_mqtt
docker buildx build . -t $THE_REGISTRY/$THE_IMAGE --platform linux/arm/v7 --load
docker push $THE_REGISTRY/$THE_IMAGE
```

## Build (Native)

```
THE_REGISTRY=ghcr.io/ussjoin
THE_IMAGE=weft_mqtt
docker build -t $THE_REGISTRY/$THE_IMAGE .
docker push $THE_REGISTRY/$THE_IMAGE
```

## Config

There isn't actually as much config as you'd think---Mosquitto's defaults are pretty close---but anyway, look at [files/mosquitto.conf](files/mosquitto.conf) for the Mosquitto config file. It's placed in at build time.

## Run

```
THE_REGISTRY=ghcr.io/ussjoin
docker run -d --rm \
    -p 1883:1883 \
    -p 9001:9001 \
    $THEREGISTRY/weft_mqtt
```

If you want to check on Mosquitto while it runs:
`docker exec -ti <containerid> /bin/sh`

### How to check the broker

If you just want to test that the broker is working, this will push some messages to it. Adjust the `-h` if not running it inside the broker's own container.

```
while true;
  do mosquitto_pub -h localhost -p 1883 -t weft/test -m "The time is now `date` and today's word is `shuf -n1 /usr/share/dict/words`";
  sleep 3;
done
```



