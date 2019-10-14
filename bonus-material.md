# Bonus Material
If you completed the previous exercises and still have time, move on to bonus material!

## Exercise 7 - Configure Load Balancing
Now that there are multiple instances of your pods, you will want load balancing between them. This way a client can call a single hostname and will be routed to a healthy pod replica. This is done in CaaS though a Service object and a Route object.

The Service object is a load balancer. The Service appears as a single IP to clients and load balances inbound traffic to multiple endpoints where your pod instances are the endpoints.

The Route object exposes a Service object as an externally-reachable hostname.

You'll create the Service object first. Review the `service.yaml` manifest file.

Again, in the `metadata` section, edit the Service's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-service` and `jpotte46-app`. Also, in the `spec` section, edit the value of `selector.app` by replacing `glits` with your CDSID.

Note the value of `selector.app` should equal the value of your Pods' labels `app`. This will associate this Service with those Pods causing inbound traffic to the Service to be forwarded to the Pods. Save the file.

```
# Create the service
oc create -f ./service.yaml

# Note the IP address of your service
oc get services
    NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)
    YOUR_SERVICE_NAME        ClusterIP   19.2.46.66    <none>        8080/TCP

# Curl your service's IP address on port 8080 at /api/v1/hello
curl 19.2.46.66:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.28.230"},"error":null}

# Run the same curl a few times and see that responses come back from each pod.
curl 19.2.46.66:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.29.59"},"error":null}
curl 19.2.46.66:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.28.230"},"error":null}
```
## Exercise 8

Now that your have a single IP address for your app, you will want to provide an externally-reachable hostname. At Ford, we do this with an OpenShift Route object.

Review the `route.yaml` manifest file.

In the `metadata` section, edit the Route's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-route` and `jpotte46-app`.

In the `spec` section, edit the value of `host` by replacing `glits` with your CDSID. For example, I use `host: jpotte46-app.app.caas.ford.com`. This causes traffic sent to this DNS hostname to be service by this Route object.

In the `spec` section, edit the value of `to.name` by replacing `glits` with your CDSID. Note the value of `to.name` should equal the value of your Service's `name`. This will associate this Route with your Service causing inbound traffic to the Service to be forwarded to the Pods. Save the file.

```
# Create the route
oc create -f route.yaml

oc get routes

# Curl the route a few times using your route name
# Now use HTTPS and not port 8080
curl https://jpotte46-app.app.caas.ford.com/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.29.59"},"error":null}
curl https://jpotte46-app.app.caas.ford.com/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.28.230"},"error":null}
```

## Exercise 9
deploy a new version without downtime
