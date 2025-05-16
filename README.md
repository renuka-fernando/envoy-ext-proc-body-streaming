# envoy-ext-proc-body-streaming

POC on using Envoy External Processor (ext proc) to process body in streaming mode.

Please update the netty-echo image in [docker-compose.yaml](docker-compose.yaml) to the image that matches your local environment architecture.

![Request flow](./resources/request_flow.png)


## Full Duplex Streaming

### Run Ext Proc Service

```sh
cd external_processor-full-duplex-stream;
go run main.go -write_data_to_file
```

### Start Envoy Docker Compose

```sh
docker compose down; docker compose up -d; docker compose logs -ft
```

### Check Memory Consumption

```sh
docker compose stats envoy
```

### Test the Ext Proc Service

#### Full Duplex Streaming
```sh
for i in {1..10}; do
    curl --location 'http://localhost:18080/full-duplex-streamed' -vvv \
    --header 'Content-Type: video/mp4' \
    --data-binary '@./resources/i_want_it_that_way.mp4' -o "./temp/response-${i}.mp4" &;
done
```

```log
CONTAINER ID   NAME                                         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O    PIDS
001dd5d9a9a8   envoy-ext-proc-body-streaming-envoy-1        26.25%    696.1MiB / 1GiB       67.98%    1.65GB / 1.47GB   0B / 0B      12
cfda2f872e3c   envoy-ext-proc-body-streaming-echo-netty-1   99.34%    974.2MiB / 3.814GiB   24.94%    625MB / 129MB     0B / 377kB   18
```

#### Streaming

```sh
for i in {1..10}; do
    curl --location 'http://localhost:18080/streamed' -vvv \
    --header 'Content-Type: video/mp4' \
    --data-binary '@./resources/i_want_it_that_way.mp4' -o "./temp/response-${i}.mp4" &;
done
```

```log
CONTAINER ID   NAME                                         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O    PIDS
001dd5d9a9a8   envoy-ext-proc-body-streaming-envoy-1        26.25%    696.1MiB / 1GiB       67.98%    1.65GB / 1.47GB   0B / 0B      12
cfda2f872e3c   envoy-ext-proc-body-streaming-echo-netty-1   99.34%    974.2MiB / 3.814GiB   24.94%    625MB / 129MB     0B / 377kB   18
```
