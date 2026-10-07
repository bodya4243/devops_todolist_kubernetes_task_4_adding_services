# Testing the ToDo application services

These commands assume that `kubectl` is configured for the target cluster. The
application and both Services are in the `my-namespace` namespace.

## Apply the resources

From the repository's `.infrastructure` directory, create the namespace and
application Pods, then create the Services:

```powershell
kubectl apply -f namespace.yml
kubectl apply -f todoapp-pod.yml
kubectl apply -f services/clusterIp.yml
kubectl apply -f services/nodeport.yml
kubectl get pods,svc -n my-namespace
```

Wait until the ToDo Pods report `Running` and `READY 1/1` before testing.

## Call the ClusterIP Service from BusyBox

The ClusterIP Service is named `kube2py-service` and exposes port `80`.
The BusyBox Pod is in the same `my-namespace` namespace as the Service, so use
the Service DNS name:

```powershell
kubectl apply -f busybox.yml
kubectl wait --for=condition=Ready pod/busybox -n my-namespace --timeout=60s
kubectl exec -n my-namespace busybox -- curl -i http://kube2py-service.my-namespace.svc.cluster.local/
```

An HTTP response from the ToDo application confirms that in-cluster DNS and
the ClusterIP Service are working. The shorter name
`kube2py-service.my-namespace` is also resolvable from the BusyBox Pod.

## Access the application with port-forward

Forward a local port to the ClusterIP Service:

```powershell
kubectl port-forward -n my-namespace service/kube2py-service 8080:80
```

Keep that command running and open [http://localhost:8080](http://localhost:8080)
in a browser, or test it in a second terminal:

```powershell
curl.exe -i http://localhost:8080/
```

Press `Ctrl+C` in the terminal running `port-forward` when finished.

## Access the application through the NodePort Service

The NodePort Service `kube2py-nodeport-service` listens on port `30007` on
every cluster node. Find a node address:

```powershell
kubectl get nodes -o wide
```

Open `http://<NODE-IP>:30007/` in a browser, replacing `<NODE-IP>` with the
node's reachable `INTERNAL-IP` or `EXTERNAL-IP`. For example:

```text
http://192.168.1.10:30007/
```

For local clusters, use the address provided by that cluster (for example,
`minikube ip` for Minikube). Ensure that port `30007` is reachable from the
machine where you run the browser.
