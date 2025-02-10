# OpenShift 4.15 Labs - Part 1 

## Introduction
This repository contains a set of labs for students learning OpenShift 4.15. Each student will use a single shared namespace, so all resource names should be uniquely prefixed with their username.

### OpenShift Console: `console-openshift-console.apps-crc.testing`
- change the console name based on your openshift

---

## Lab 1: Logging into OpenShift
### Objective:
Learn how to log into the OpenShift cluster using the CLI.

### Steps:
1. Open a terminal.
2. Login using the `oc` command:
   ```sh
   oc login -u <your-username> -p <your-password> --server=https://api.crc.testing:6443
   ```
3. Verify that you are logged in:
   ```sh
   oc whoami
   ```
4. Set the namespace for your work:
   ```sh
   oc project trainingdev
   ```

---

## Lab 2: Tagging and Pushing an Image to OpenShift Registry
### Objective:
Use Docker Desktop to tag an image and push it to the OpenShift internal registry.

### Steps:
1. Get the OpenShift registry login token:
   ```sh
   oc whoami -t
   ```
2. Login to the OpenShift registry:
   ```sh
   docker login -u <your-username> -p $(oc whoami -t) default-route-openshift-image-registry.apps-crc.testing
   ```
3. Tag the image (using a unique tag, e.g., `myapp-<username>`):
   ```sh
   docker tag myapp:v1 default-route-openshift-image-registry.apps-crc.testing/trainingdev/myapp-<username>:v1
   ```
4. Push the image:
   ```sh
   docker push default-route-openshift-image-registry.apps-crc.testing/trainingdev/myapp-<username>:v1
   ```

---

## Lab 3: Deploying the Image in OpenShift
### Objective:
Run the image as a deployment in OpenShift.

### Steps:
1. Create a deployment using your pushed image:
   ```sh
   oc new-app --image=default-route-openshift-image-registry.apps-crc.testing/trainingdev/myapp-<username>:v1 --name=myapp-<username>
   ```
2. Verify the deployment:
   ```sh
   oc get pods
   ```

---

## Lab 4: Exposing the Deployment
### Objective:
Expose the deployment to be accessible over the network.

### Steps:
1. Create a service for the deployment:
   ```sh
   oc expose deployment myapp-<username> --port=8080
   ```
2. Expose the service as a route:
   ```sh
   oc expose svc myapp-<username>
   ```
3. Get the URL of the exposed application:
   ```sh
   oc get routes
   ```

---

## Lab 5: Scaling the Deployment
### Objective:
Scale the application up and down.

### Steps:
1. Scale up the deployment to 3 replicas:
   ```sh
   oc scale deployment myapp-<username> --replicas=3
   ```
2. Verify the scaling:
   ```sh
   oc get pods
   ```
3. Scale down to 1 replica:
   ```sh
   oc scale deployment myapp-<username> --replicas=1
   ```

---

## Notes
- Ensure all resource names include `-<username>` to prevent conflicts in the shared namespace.
- Use `oc delete` if you need to clean up resources after each lab:
  ```sh
  oc delete all -l app=myapp-<username>
  
