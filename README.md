# NodeJS Odigos demo

A super simple demo showing OTel setup with Odigos and a locally built app.

## Instructions
Set up a Kubernetes cluster and install Odigos with Helm:
```
kind create cluster

helm repo add odigos https://keyval-dev.github.io/odigos-charts/
helm install my-odigos odigos/odigos --namespace odigos-system --create-namespace
```

Build the image referenced in the `odigos-node.yaml` file:
```
docker build . -t odigos-node
```

Load the locally built image to make it available in the cluster without pulling (`imagePullPolicy: Never`):
```
kind load docker-image odigos-node:latest
```

Start the app and Jaeger on Kube:
```
kubectl apply -f odigos-node.yaml,jaeger.yaml
```

Expose Odigos UI:
```
kubectl port-forward svc/odigos-ui 3000:3000 -n odigos-system
```

Go to http://localhost:3000. You'll be asked to choose the target application - select `odgios-node` and `Save changes`. Next, choose the destination - for the purposes of this demo, we'll go with Jaeger (under `Self-hosted`). Name the destination whatever you want, but make sure to use `jaeger.default:4317` as the URL - `jaeger` is the name of the Kubernetes service and it lives in the `default` namespace. `4317` is the port used to send traces over gRPC.

With the destination saved, we can proceed with generating traces:
```
kubectl port-forward svc/odigos-node 3333:3333
curl -X GET http://localhost:3333
```

Make Jaeger UI accessible from `localhost`:
```
kubectl port-forward svc/jaeger 16686:16686
```

Finally, go to http://localhost:16686 and check the traces from the `odigos-node` service!

## Cleanup

```
kind delete cluster
```

## Troubleshooting

Check `kubectl logs svc/odigos-gateway -n odigos-system` to ensure no Jaeger errors.
To check if Odigos works, try another destination, e.g. Honeycomb.
