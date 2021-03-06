# gRPC Ping

A simple gRPC server written in Go that you can use for testing.

This sample is dependent on
[this issue](https://github.com/knative/serving/issues/1047) to be complete.

## Prerequisites

1. [Install Knative](https://github.com/knative/docs/blob/master/install/README.md)
1. Install [docker](https://www.docker.com/)

## Build and run the gRPC server

Build and run the gRPC server. This command will build the server and use
`kubectl` to apply the configuration.

```shell
REPO="gcr.io/<your-project-here>"

# Build and publish the container, run from the root directory.
docker build \
  --tag "${REPO}/serving/samples/grpc-ping-go" \
  --file=serving/samples/grpc-ping-go/Dockerfile .
docker push "${REPO}/serving/samples/grpc-ping-go"

# Replace the image reference with our published image.
perl -pi -e "s@github.com/knative/docs/serving/samples/grpc-ping-go@${REPO}/serving/samples/grpc-ping-go@g" serving/samples/grpc-ping-go/*.yaml

# Deploy the Knative sample.
kubectl apply --filename serving/samples/grpc-ping-go/sample.yaml
```

## Use the client to stream messages to the gRPC server

1. Fetch the created ingress hostname and IP.

```shell
# Put the Host name into an environment variable.
export SERVICE_HOST=`kubectl get route grpc-ping --output jsonpath="{.status.domain}"`

# Put the ingress IP into an environment variable.
export SERVICE_IP=`kubectl get svc knative-ingressgateway --namespace istio-system --output jsonpath="{.status.loadBalancer.ingress[*].ip}"`
```

1. Use the client to send message streams to the gRPC server.

```shell
go run -tags=grpcping ./serving/samples/grpc-ping-go/client/client.go -server_addr="$SERVICE_IP:80" -server_host_override="$SERVICE_HOST"
```
