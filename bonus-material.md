# Bonus Material
If you completed the previous exercises and still have time, move on to bonus material!

## Exercise 7 - Configure Load Balancing
Now that there are multiple instances of your pods, you will want load balancing between them. This way a client can call a single hostname and will be routed to a healthy pod replica. This is done in CaaS though a Service object and a Route object.

The Service object is a load balancer. The Service appears as a single IP to clients and load balances inbound traffic to multiple endpoints where your pod instances are the endpoints.

The Route object exposes a Service object as an externally-reachable hostname.

You'll create the Service object first. Review the `service.yaml` manifest file.

Again, in the `metadata` section, edit the Service's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-service` and `jpotte46-app`. Also, in the `spec` section, edit the value of `selector.app` by replacing `glits` with your CDSID.

Note the value of `selector.app` should equal the value of your Pods' `app` label. Configuring the selector this way will associate this Service with those Pods. Inbound traffic to this Service will be forwarded to associated Pods. **Save the file.**

```
# Create the service
oc create -f ./service.yaml

# Note the IP address of your service
oc get service YOUR_SERVICE_NAME
    NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)
    YOUR_SERVICE_NAME        ClusterIP   19.2.46.66    <none>        8080/TCP

# Curl your service's IP address at /api/v1/hello. Replace IP below with your service's IP address.
export SERVICE_IP=19.2.46.66  # <-------- YOUR SERVICE IP ADDRESS FROM ABOVE
curl ${SERVICE_IP}:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.28.230"},"error":null}

# Run the same curl a few times and see that responses come back from each pod.
while true; do
  curl ${SERVICE_IP}:8080/api/v1/hello;
  echo "";
  sleep 1;
done

# Stop loop with Ctrl+C.

```
## Exercise 8

Now that your have a single IP address for your app, you will want to provide an externally-reachable hostname. At Ford, we do this with an OpenShift Route object.

Review the `route.yaml` manifest file.

In the `metadata` section, edit the Route's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-route` and `jpotte46-app`.

In the `spec` section, edit the value of `host` by replacing `glits` with your CDSID. For example, I use `host: jpotte46-app.app.caas.ford.com`. This causes traffic sent to this hostname to be serviced by this Route object.

In the `spec` section, edit the value of `to.name` by replacing `glits` with your CDSID. Note the value of `to.name` should equal the value of your Service's `name`. This will associate this Route with your Service. **Save the file.**

```
# Create the route
oc create -f route.yaml

oc get routes

# Curl the route a few times using YOUR route name
# Note we use HTTPS now (and not port 8080)
export YOUR_HOST=jpotte46-app.app.caas.ford.com # <------- YOUR HOSTNAME
while true; do
  curl https://${YOUR_HOST}/api/v1/hello;
  echo "";
  sleep 1;
done

# Stop loop with Ctrl+C.

```

Delete everything.

```
oc delete -f deployment.yaml
oc delete -f service.yaml
oc delete -f route.yaml
```
