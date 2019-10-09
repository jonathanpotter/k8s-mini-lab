# Getting Your Hands-On CaaS

This is a hands-on lab where you will learn the basics of deploying containerized applications on the CaaS platform.  We’ll provide a live CaaS environment where you can deploy something small, scale it up, then deploy a new version without downtime.

## Pre-requisites
- Laptop with Ford GitHub access.
- Familiarity with command-line activities (moving files, setting `PATH` environment variable, cloning a git repo, etc...)

## Exercise 1 - Clone the Lab Repo
Clone the [glits-lab](https://github.ford.com/JPOTTE46/glits-lab) git repo.

Cheatsheet:
```
cd ~/workspace
git clone git@github.ford.com:JPOTTE46/glits-lab.git
cd ./glits-lab
```

## Exercise 2 - Install RedHat OpenShift Command-Line Interface
You need to install the `oc` cli in order to issue commands to CaaS.

- Download version `3.11.135` from https://files.caas.ford.com:9443/oc-cli/. Select the file which name matches your system: Windows, MacOS, or Linux.
- For Windows, unzip the downloaded file and copy the executable binary `/bin/oc.exe` somewhere on your system.
- For MacOS/Linux, you can put `oc` into `/usr/local/bin` with `tar -xvzf ~/Downloads/oc-3.11.135-macosx.tar.gz -C /usr/local/bin`.

Test that you are configured correctly by running `oc version` from the terminal. It should look similar to:

```
oc version
    oc v3.11.135
    kubernetes v1.11.0+d4cacc0
    features: Basic-Auth

    Server https://api.caas.ford.com:443
    openshift v3.11.135
    kubernetes v1.11.0+d4cacc0
```

## Exercise 3 - Authenticate to CaaS
Authenticate to Ford's CaaS platform with `oc`.

You have been granted temporary access to an environment on Ford's CaaS platform. You can login with your CDSID and password.

```
oc login https://api.caas.ford.com
# Your username
# Your password

# Check that you are targeting the devenablement-dev project. You should see an "*".
oc projects
```

If you are using GitBash on Windows, the `oc` cli has a bug where it outputs your password to the terminal. You can avoid this with the `winpty` utility replacing the command above with `winpty oc login https://api.caas.ford.com`.

## Review of Ford's Container Image Registry
Ford's Container Image Registry is at https://registry.ford.com. For this lab, I have created a container image and uploaded it there to a repository at https://registry.ford.com/repository/jpotte46/springboot-hello-world. The image is of a minimal "Hello World" web service. The image is a binary image that contains all dependencies required to run the app. You can run the image locally with a tool like `docker`, or you can deploy this container image to CaaS.

## Exercise 4 - Create a Pod
You will deploy the container image to CaaS using a Pod object type. Review the `pod.yaml` manifest file.

In the `metadata` section, edit the Pod's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-pod` and `jpotte46-app`. Save the file.

```
# Deploy your pod.
oc create -f ./pod.yaml

# List app pods.
oc get pods

# View the events of your pod.
oc describe pod YOUR_POD_NAME

# View the app logs of your pod.
oc logs -f YOUR_POD_NAME
```

It will take a couple minutes for your app to start. Keep checking the logs until you the message `Tomcat started on port(s): 8080 (http) with context path ''`. The end the log streaming with Ctrl+C command.

Then test that your app is responding.

```
# Get your pod's IP address
oc describe pod YOUR_POD_NAME | grep IP
    IP: 19.2.17.116

# Curl your pod's IP address on port 8080 at /api/v1/hello
curl 19.2.17.116:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.17.116"},"error":null}

# Delete the pod
oc delete pod YOUR_POD_NAME
```

## Exercise 5 - Create a Deployment
Pods can fail. If they do, they will not recover themselves; however, CaaS will automatically recover from a Pod failure using a Deployment controller object. With a Deployment, you declaratively specify your desired state, and CaaS will continually monitor current state and make changes as needed to maintain your desired state.

Review the `deployment.yaml` manifest file.

Again, in the `metadata` section, edit the Deployment's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-deployment` and `jpotte46-app`. Also, in the `spec` section, edit the values of `selector.matchLabels.app` and `template.metadata.labels.app` replace `glits` with your CDSID. Save the file.

```
# Create the deployment
oc create -f ./deployment.yaml

oc get deployments

oc get pods

# Get your pod's IP address
oc describe pod YOUR_POD_NAME | grep IP
    IP: 19.2.28.230

# Curl your pod's IP address on port 8080 at /api/v1/hello
curl 19.2.28.230:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.28.230"},"error":null}

# Delete a running pod and CaaS will recreate it.
oc delete pod YOUR_POD_NAME
oc get pods
```

CaaS will start a new pod to replace the one that you deleted.

## Exercise 6 - Scale Up for High Availability
In the last exercise, you specified only 1 pod instance, so there was a service outage during the time between deletion of the old pod and startup of a replacement pod. To provide greater availability, you always want to run 2 or more instances in production. Kubernetes calls pod instances replicas.

```
# Scale up the deployment from 1 pod to 2 pods
oc scale --replicas=2 deployment YOUR_DEPLOYMENT_NAME

oc get pods
```

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
