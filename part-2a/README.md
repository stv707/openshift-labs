## Part 2a: Advanced OpenShift Operations

# OpenShift YAML Deployment Exercise

## Introduction

In this exercise, you will learn how to deploy applications in OpenShift using YAML files. You will create the following resources:

- **Deployment**: Defines and manages your application instances.
- **Pods**: Runs your application containers.
- **Service**: Exposes your application internally within OpenShift.
- **Route**: Exposes your application externally via OpenShift’s router.

### Prerequisites

- Logged into an OpenShift cluster with `oc` CLI.
- A namespace/project where you have permissions to create resources.
- Modify all resources to replace `<username>` with your actual username to ensure uniqueness.

---

## Step 1: Create the Deployment

Create a YAML file called **deployment.yaml** with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia-<username>
  labels:
    app: kubia-<username>
spec:
  replicas: 2
  selector:
    matchLabels:
      app: kubia-<username>
  template:
    metadata:
      labels:
        app: kubia-<username>
    spec:
      containers:
        - name: kubia
          image: stv707/kubia:v1
          ports:
            - containerPort: 8080
```

### Apply the Deployment

Run the following command to deploy:

```sh
oc apply -f deployment.yaml
```

Verify the deployment:

```sh
oc get deployments
```

---

## Step 2: Create the Service

Create a YAML file called **service.yaml** with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubia-<username>
  labels:
    app: kubia-<username>
spec:
  selector:
    app: kubia-<username>
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### Apply the Service

Run the following command to create the service:

```sh
oc apply -f service.yaml
```

Verify the service:

```sh
oc get services
```

---

## Step 3: Create the Route

Create a YAML file called **route.yaml** with the following content:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: kubia-<username>
  labels:
    app: kubia-<username>
spec:
  to:
    kind: Service
    name: kubia-<username>
  port:
    targetPort: 8080
  tls:
    termination: edge
  wildcardPolicy: None
```

### Apply the Route

Run the following command to create the route:

```sh
oc apply -f route.yaml
```

Verify the route:

```sh
oc get routes
```

---

## Step 4: Verify Application Deployment

Once all resources are created, check if your application is running correctly.

### Check the Pods

```sh
oc get pods
```

### Check the Route URL

Run the following command to get the external URL:

```sh
oc get route kubia-<username> -o jsonpath='{.spec.host}'
```

Open the displayed URL in your browser to verify the deployment.

---

## Cleanup (Optional)

If you want to remove all created resources, run the following commands:

```sh
oc delete deployment kubia-<username>
oc delete service kubia-<username>
oc delete route kubia-<username>
```

---

## Conclusion

You have successfully deployed an application in OpenShift using YAML files! 🎉

Modify `<username>` with your actual username in all YAML files to ensure uniqueness when deploying in a shared namespace.

---