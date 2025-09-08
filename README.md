# envoy-ext-proc-body-streaming

POC on using Envoy External Processor (ext proc) to process body in streaming mode.

Please update the netty-echo image in [docker-compose.yaml](docker-compose.yaml) to the image that matches your local environment architecture.

![Request flow](./resources/request_flow.png)

## 1. Full Duplex Streaming Mode

### Start Envoy Docker Compose

```sh
mkdir -p envoy/log
docker compose down; docker compose up -d; docker compose logs -ft envoy
```

### Run Ext Proc Service

```sh
cd external-processor-full-duplex-stream;
go run main.go -write_data_to_file
```

### Check Memory Consumption

```sh
docker compose stats envoy
```

### Start Heap Profiling (If the image renukafernando/envoy-gperftools:v1.34.1 is used)

```sh
curl 'http://localhost:9901/heapprofiler' \
  --data-raw 'enable=y'
```

### Test the Ext Proc Service

```sh
for i in {1..10}; do
    curl --location 'http://localhost:18080/full-duplex-streamed' -vvv \
    --header 'Content-Type: video/mp4' \
    --data-binary '@./resources/i_want_it_that_way.mp4' -o "./temp/response-${i}.mp4" &;
done
```

```log
CONTAINER ID   NAME                                    CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O   PIDS
334709c09931   envoy-ext-proc-body-streaming-envoy-1   0.82%     527.6MiB / 1GiB     51.52%    2.52GB / 2.49GB   0B / 0B     12
```

### Stop Heap Profiling

```sh
curl 'http://localhost:9901/heapprofiler' \
  --data-raw 'enable=n'
```

### Clean Up

1.  Stop the ext proc service and remove the temp files.
2.  Stop the Docker Compose.
    ```sh
    docker compose down
    ```

### Heap Profile

```
export PPROF_BINARY_PATH="./lib"
pprof -svg -inuse_space -output=heap_profile-inuse_space.svg envoy-v1.34.1-gperftools/envoy-static envoy/log/envoy.prof.*
pprof -svg -inuse_objects -output=heap_profile-inuse_objects.svg envoy-v1.34.1-gperftools/envoy-static envoy/log/envoy.prof.*
pprof -svg -alloc_space -output=heap_profile-alloc_space.svg envoy-v1.34.1-gperftools/envoy-static envoy/log/envoy.prof.*
pprof -svg -alloc_objects -output=heap_profile-alloc_objects.svg envoy-v1.34.1-gperftools/envoy-static envoy/log/envoy.prof.*

pprof -text -inuse_space envoy-v1.34.1-gperftools/envoy-static envoy/log/envoy.prof.*
```

Results

```sh
Some binary filenames not available. Symbolization may be incomplete.
Try setting PPROF_BINARY_PATH to the search path for local binaries.
File: envoy-static
Type: inuse_space
Showing nodes accounting for 2.47GB, 99.23% of 2.49GB total
Dropped 1097 nodes (cum <= 0.01GB)
      flat  flat%   sum%        cum   cum%
    2.43GB 97.84% 97.84%     2.43GB 97.84%  Envoy::Buffer::Slice::newStorage
    0.02GB  0.81% 98.64%     0.02GB  0.81%  Envoy::Buffer::Slice::Slice
    0.01GB  0.59% 99.23%     0.01GB  0.59%  std::allocator_traits::allocate
         0     0% 99.23%     0.02GB  0.81%  Envoy::Buffer::OwnedImpl::add
         0     0% 99.23%     0.02GB  0.81%  Envoy::Buffer::OwnedImpl::addImpl
         0     0% 99.23%     2.43GB 97.84%  Envoy::Buffer::OwnedImpl::reserveSingleSlice
         0     0% 99.23%     1.70GB 68.47%  Envoy::Event::DispatcherImpl::createFileEvent()::{lambda(unsigned int)#2}::operator()
         0     0% 99.23%     0.02GB  0.72%  Envoy::Event::FileEventImpl::assignEvents()::{lambda(int, short, void*)#3}::_FUN
         0     0% 99.23%     0.02GB  0.74%  Envoy::Event::FileEventImpl::assignEvents()::{lambda(int, short, void*)#3}::operator()
         0     0% 99.23%     0.02GB  0.75%  Envoy::Event::FileEventImpl::mergeInjectedEventsAndRunCb
         0     0% 99.23%     2.44GB 97.99%  Envoy::Extensions::Common::ExternalProcessing::ProcessorClientImpl::sendRequest
         0     0% 99.23%     2.44GB 97.99%  Envoy::Extensions::Common::ExternalProcessing::ProcessorStreamImpl::send
         0     0% 99.23%     1.69GB 67.85%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::decodeData
         0     0% 99.23%     0.76GB 30.40%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::encodeData
         0     0% 99.23%     2.44GB 98.33%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::handleDataFullDuplexStreamedMode
         0     0% 99.23%     2.44GB 98.33%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::handleDataStreamedModeBase
         0     0% 99.23%     2.44GB 98.33%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::onData
         0     0% 99.23%     2.44GB 97.98%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::sendBodyChunk
         0     0% 99.23%     2.44GB 97.99%  Envoy::Extensions::HttpFilters::ExternalProcessing::Filter::sendRequest
         0     0% 99.23%     2.44GB 97.99%  Envoy::Grpc::AsyncStream::sendMessage
         0     0% 99.23%     0.01GB  0.57%  Envoy::Grpc::AsyncStreamCallbacks::onReceiveMessageRaw
         0     0% 99.23%     0.02GB  0.64%  Envoy::Grpc::AsyncStreamImpl::onData
         0     0% 99.23%     2.43GB 97.84%  Envoy::Grpc::Common::serializeMessage
         0     0% 99.23%     2.44GB 97.99%  Envoy::Grpc::Internal::sendMessageUntyped
         0     0% 99.23%     0.77GB 31.03%  Envoy::Http::ActiveStreamDecoderFilter::encodeData
         0     0% 99.23%     0.02GB  0.64%  Envoy::Http::AsyncStreamImpl::encodeData
         0     0% 99.23%     0.76GB 30.46%  Envoy::Http::CodecClient::CodecReadFilter::onData
         0     0% 99.23%     0.76GB 30.46%  Envoy::Http::CodecClient::onData
         0     0% 99.23%     1.69GB 67.85%  Envoy::Http::ConnectionManagerImpl::ActiveStream::decodeData
         0     0% 99.23%     1.69GB 67.95%  Envoy::Http::ConnectionManagerImpl::onData
         0     0% 99.23%     1.69GB 67.96%  Envoy::Http::FilterManager::decodeData
         0     0% 99.23%     0.77GB 31.03%  Envoy::Http::FilterManager::encodeData
         0     0% 99.23%     0.01GB  0.54%  Envoy::Http::Http1::BalsaParser::execute
         0     0% 99.23%     0.75GB 30.20%  Envoy::Http::Http1::ClientConnectionImpl::dispatch
         0     0% 99.23%     0.76GB 30.39%  Envoy::Http::Http1::ClientConnectionImpl::onBody
         0     0% 99.23%     2.44GB 98.36%  Envoy::Http::Http1::ConnectionImpl::dispatch
         0     0% 99.23%     2.44GB 98.23%  Envoy::Http::Http1::ConnectionImpl::dispatchBufferedBody
         0     0% 99.23%     0.01GB  0.54%  Envoy::Http::Http1::ConnectionImpl::dispatchSlice
         0     0% 99.23%     1.69GB 67.94%  Envoy::Http::Http1::ServerConnectionImpl::dispatch
         0     0% 99.23%     1.69GB 67.84%  Envoy::Http::Http1::ServerConnectionImpl::onBody
         0     0% 99.23%     0.01GB  0.51%  Envoy::Http::Http2::ConnectionImpl::Http2Visitor::OnDataForStream
         0     0% 99.23%     0.01GB  0.51%  Envoy::Http::Http2::ConnectionImpl::StreamImpl::decodeData
         0     0% 99.23%     0.01GB  0.51%  Envoy::Http::Http2::ConnectionImpl::onBeginData
         0     0% 99.23%     0.77GB 30.91%  Envoy::Http::ResponseDecoderWrapper::decodeData
         0     0% 99.23%     1.70GB 68.55%  Envoy::Network::ConnectionImpl::ConnectionImpl()::{lambda(unsigned int)#8}::operator()
         0     0% 99.23%     1.71GB 68.71%  Envoy::Network::ConnectionImpl::onFileEvent
         0     0% 99.23%     2.45GB 98.39%  Envoy::Network::ConnectionImpl::onRead
         0     0% 99.23%     1.71GB 68.69%  Envoy::Network::ConnectionImpl::onReadReady
         0     0% 99.23%     2.45GB 98.41%  Envoy::Network::FilterManagerImpl::onContinueReading
         0     0% 99.23%     2.45GB 98.40%  Envoy::Network::FilterManagerImpl::onRead
         0     0% 99.23%     0.77GB 31.03%  Envoy::Router::Filter::onUpstreamData
         0     0% 99.23%     0.77GB 30.91%  Envoy::Router::UpstreamCodecFilter::CodecBridge::decodeData
         0     0% 99.23%     0.77GB 31.03%  Envoy::Router::UpstreamRequest::decodeData
         0     0% 99.23%     0.77GB 31.03%  Envoy::Router::UpstreamRequestFilterManagerCallbacks::encodeData
         0     0% 99.23%     0.01GB  0.53%  event_persist_closure
         0     0% 99.23%     0.01GB  0.51%  http2::Http2DecoderAdapter::OnDataPayload
         0     0% 99.23%     0.01GB  0.51%  http2::Http2DecoderAdapter::ProcessInput
         0     0% 99.23%     0.01GB  0.51%  http2::Http2DecoderAdapter::ProcessInputFrame
         0     0% 99.23%     0.01GB  0.51%  http2::Http2FrameDecoder::DecodeFrame
         0     0% 99.23%     0.01GB  0.51%  http2::Http2TraceLogger::OnStreamFrameData
         0     0% 99.23%     0.01GB  0.51%  http2::adapter::EventForwarder::OnStreamFrameData
         0     0% 99.23%     0.01GB  0.51%  http2::adapter::OgHttp2Session::OnStreamFrameData
         0     0% 99.23%     0.01GB  0.53%  quiche::BalsaFrame::ProcessInput
         0     0% 99.23%     1.71GB 68.60%  std::_Function_handler::_M_invoke
         0     0% 99.23%     1.71GB 68.61%  std::__invoke_impl
         0     0% 99.23%     1.71GB 68.60%  std::__invoke_r
         0     0% 99.23%     1.71GB 68.60%  std::function::operator()
```

## 2. Streaming Mode

### Start Envoy Docker Compose

```sh
docker compose down; docker compose up -d; docker compose logs -ft envoy
```

### Run Ext Proc Service

```sh
cd external-processor-stream;
go run main.go -write_data_to_file
```

### Check Memory Consumption

```sh
docker compose stats envoy
```

### Test the Ext Proc Service

```sh
for i in {1..10}; do
    curl --location 'http://localhost:18080/streamed' -vvv \
    --header 'Content-Type: video/mp4' \
    --data-binary '@./resources/i_want_it_that_way.mp4' -o "./temp/response-${i}.mp4" &;
done
```

```log
CONTAINER ID   NAME                                    CPU %     MEM USAGE / LIMIT   MEM %     NET I/O           BLOCK I/O   PIDS
f92bccc9c103   envoy-ext-proc-body-streaming-envoy-1   42.67%    151.7MiB / 1GiB     14.82%    4.56GB / 5.43GB   0B / 0B
```
