# Getting Your Hands-On CaaS

This is a hands-on lab where you will learn the basics of deploying containerized applications on the CaaS platform.  We’ll provide a live CaaS environment where you can deploy something small, scale it up, then deploy a new version without downtime.

## Pre-requisites
- Laptop with Ford GitHub access and permissions to download/run an `.exe` file.
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

- Download version `3.11.135` from https://files.caas.ford.com:9443/oc-cli/. Select the file with the name matching your system: Windows, MacOS, or Linux.
- For Windows
  - Open the downloaded zip file.
  - Copy the executable binary `oc.exe` somewhere on your system.
  - Optionally, you can set your `PATH` environment variable to include the directory where you saved `oc.exe`.
- For MacOS/Linux
  - Put `oc` into `/usr/local/bin` with `tar -xvzf ~/Downloads/oc-3.11.135-macosx.tar.gz -C /usr/local/bin`.
  - Optionally, you can set your `PATH` environment variable to include `/usr/local/bin`.

Test that you are configured correctly by opening a terminal (GitBash, cmd, PowerShell, iTerm2, etc...) and running `oc version` from the terminal.

If you have not set the `PATH` env variable, you'll need to use the full path to the location of `oc.exe`, for example `C:\Users\jpotte46\Desktop\oc.exe version`.

Your output should look similar to:

```
oc version
    oc v3.11.135
    kubernetes v1.11.0+d4cacc0
    features: Basic-Auth
```

## Exercise 3 - Authenticate to CaaS
Authenticate to Ford's CaaS platform with `oc`.

You have been granted temporary access to an environment on Ford's CaaS platform. You can login with your CDSID and password. Then target the `devenablement-dev` project.

```
# Login to CaaS.
# If you get "error: Service Unavailable", then you have a proxy error. Raise your hand for help.
oc login https://api.caas.ford.com
# Your username
# Your password

# Target the devenablement-dev project. You will see an "*" next to the targeted project.
oc project devenablement-dev
```

If you are using GitBash on Windows, the `oc` cli has a bug where it outputs your password to the terminal. You can avoid this with the `winpty` utility replacing the command above with `winpty oc login https://api.caas.ford.com`.

## Review of Ford's Container Image Registry
Ford's Container Image Registry is at https://registry.ford.com. Note that you will not have access to it until you sign up when you procure your CaaS environment. However, for this lab, I have already created a [container image](https://registry.ford.com/repository/jpotte46/springboot-hello-world) for you and uploaded it. The image contains a minimal "Hello World" web service.

## Exercise 4 - Create a Pod
You will deploy the container image to CaaS using a Pod object type. Review the `pod.yaml` manifest file.

In the `metadata` section, edit the Pod's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-pod` and `jpotte46-app`. **Save the file.**

```
# Deploy your pod.
oc create -f ./pod.yaml

# List app pods.
oc get pods

# View the events of your pod.
oc describe pod YOUR_POD_NAME

# Stream the app logs of your pod. (You can stop streaming with Ctrl+C)
oc logs -f YOUR_POD_NAME
```

It will take a couple minutes for your app to start. Keep checking the logs until you the message `Tomcat started on port(s): 8080 (http) with context path ''`.

Once the app has started, end the log streaming with Ctrl+C.

Then test that your app is responding using curl or a web browser. **Modify the curl command below to use your own pod's IP address.**

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

Again, in the `metadata` section, edit the Deployment's `name` value and the `app` label value to something unique by replacing `glits` with your CDSID. For example, I use `name: jpotte46-deployment` and `jpotte46-app`. Also, in the `spec` section, edit the values of `selector.matchLabels.app` and `template.metadata.labels.app` replace `glits` with your CDSID. **Save the file.**

```
# Create the deployment
oc create -f ./deployment.yaml

oc get deployments

# Any oc command output can be filtered by a label with -l.
# Filter the results where label app=YOUR_APP_NAME as defined in deployment.yaml.
# Continually run (or watch) the command with -w.
oc get pods -l app=YOUR_APP_NAME -w

# Run the command above until the output shows the pod status is Running. Stop the command with Ctrl+C.
# Then get your pod's IP address.
oc describe pods -l app=YOUR_APP_NAME | grep IP
    IP: 19.2.28.230

# Curl your pod's IP address on port 8080 at /api/v1/hello
curl 19.2.28.230:8080/api/v1/hello
    {"result":{"greeting":"Hello from pod 19.2.28.230"},"error":null}
```

Now open a new terminal window where you will watch the pod be deleted. CaaS will then start a new pod to replace the one that you deleted.

```
# Watch pods in new terminal.
oc get pods -l app=YOUR_APP_NAME -w

# In the original terminal, delete a running pod and CaaS will recreate it.
oc delete pod YOUR_POD_NAME
```

Once you have deleted the pod, you can close the new terminal window.

## Exercise 6 - Scale Up for High Availability
In the last exercise, you specified only 1 pod instance, so your app experienced a service outage during the time between deletion of the old pod and startup of a replacement pod. To provide greater availability, you always want to run 2 or more instances in production. (Note that Kubernetes calls pod instances "replicas" so you'll hear the words instances and replicas used interchangeably in conversation.)

```
# Scale up the deployment from 1 pod to 2 pods
oc scale --replicas=2 deployment YOUR_DEPLOYMENT_NAME

# See that the deployment now has two pods.
oc get pods -l app=YOUR_APP_NAME
```

## Free Workshop
You can learn more by attending the free CaaS Workshop. Register [online](https://it2.spt.ford.com/sites/dev/Lists/RegisterForEvent/newform.aspx). The workshop is offered monthly.
