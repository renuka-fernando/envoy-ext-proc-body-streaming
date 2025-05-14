# envoy-ext-proc-body-streaming

POC on using Envoy External Processor (ext proc) to process body in streaming mode.

Please update the netty-echo image in [docker-compose.yaml](docker-compose.yaml) to the image that matches your local environment architecture.

![Request flow](./resources/request_flow.png)

## Run Ext Proc Service

```sh
cd external_processor;
go run main.go
```

## Check Memory Consumption

```sh
docker stats
```

## Test the Ext Proc Service

```sh
for i in {1..10}; do
    curl --location 'http://localhost:18080/pets/myPetId123/history' -vvv \
    --header 'Content-Type: video/mp4' \
    --data-binary '@./resources/i_want_it_that_way.mp4' -o "./temp/response-${i}.mp4" &;
done
```
