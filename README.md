# DO-731

The objective of this repository is to investigate the `EnvoyFilter` of Istio over `v1.13` through the example [per-route config](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_authz_filter). The per-route configuration is to set the external authorzation for specific routes.

Firstly, we would start with the Envoy. In example (i.e `./envoy/envoy.yaml`), the envoy is enabled to authorize for the host `localhost:10000` but it is disabled for the host `127.0.0.1:10000`. You can try like below.

```shell
# in ./envoy
$ docker-compose up

# enabled for localhost.
$ curl -v localhost:10000

...
< HTTP/1.1 401 Unauthorized
...

# disabled for 127.0.0.1
$ curl -v 127.0.0.1:10000

...
< HTTP/1.1 200 OK
```

For Istio, it would be very similar with the configuration of Envoy. It is enabled to authorize for the public host, which comes from the outside of a cluster, and it is disabled for the internal host. 

```shell
$ cat istio/kubernetes.yaml | sed 's/NAMESPACE/YOUR_NAMESPACE/g' | sed 's/HOSTNAME/YOUR_HOSTNAME/g' | sed 's/GATEWAY/YOUR_GATEWAY.istio-system/g' | k apply -f -

$ curl -v YOUR_HOSTNAME 
...
< HTTP/1.1 401 Unauthorized
...

# Inside of the cluster
$ k run curl -ti --rm  --generator=run-pod/v1 --image=yauritux/busybox-curl --command -- sh
$ curl -v nginx.YOUR_NAMESPACE.svc.cluster.local 
...
< HTTP/1.1 200 OK
```

And if you want to check the settings of Envoy in Istio the command `istioctl pc listener POD --port 80 -o json` would be helpful.
