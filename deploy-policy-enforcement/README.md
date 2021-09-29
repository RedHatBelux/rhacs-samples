# This is a small demo to show policy enforcement with ACS

## Step 1: Building the image

Login to your favorite image repo, we will be leveraging `quay.io`:
```
podman login quay.io
podman build . -t quay.io/username/httpd-netcat:latest
podman push quay.io/username/httpd-netcat:latest
```
You will need to push twice as the first time the repo will be created, and the second time it will allow you to push.

### Step 1 alternative: Build using S2I on OpenShift

This step will leverage OpenShift to build the image. We will do this in the 'test-policy-acs' project. From this folder do the following:
```
oc new-project test-policy-acs
oc new-build --binary --strategy=docker --name httpd-netcat
oc start-build httpd-netcat --from-dir . -F
```
## Step 2: Create a policy with enforcement on ACS
We are going to create a policy that checks for the 'netcat' component in our image, as it is known that netcat can be used for malicious behaviour.

To make things easy, we can clone the "Curl in Image" policy. You can basically replace "curl" with "netcat" each step of the way. And finally, on the last step, enable enforcement at "DEPLOY" time.

Do not forget to enable the policy.

# Step 3: Deploy on OpenShift/K8s
## Step 3 with any registry
Before creating the deployment, open the `netcat-app.yml` file and change the image the one you have build in step 1.
```
kubectl apply -f netcat-app.yml
```
## Step 3 with S2I build (step 1 alternative build)
```
oc apply -f ocp-netcat-app.yaml
```

# Deployment fails due to policy
Creating the deployment should now give you an error message:
```
Error from server (Failed currently enforced policies from StackRox): error when creating "netcat-app.yml": admission webhook "policyeval.stackrox.io" denied the request:
The attempted operation violated 1 enforced policy, described below:

Policy: Netcat in image
- Description:
    ↳ Alert on deployments with netcat present
- Rationale:
    ↳ Leaving tools like netcat in an image makes it easier for attackers to
      use compromised containers
- Remediation:
    ↳ Use your package manager's "remove", "purge" or "erase" command to remove netcat
      from the image build for production containers. Ensure that any configuration
      files are also removed.
- Violations:
    - Container 'netcat-test' includes component 'netcat' (version 1.10-41.1)


In case of emergency, add the annotation {"admission.stackrox.io/break-glass": "ticket-1234"} to your deployment with an updated ticket number
```

## Variant
You can also try enabling the policy after you have created the deployment. By doing this, you will see the deployment will succeed, but any scaling up will fail. Try editing your service with `kubectl edit deployment netcat-deployment` and increase the amount of replicas. You should see an error message.
