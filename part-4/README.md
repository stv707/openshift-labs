# OpenShift Multi-Container Deployment Exercise

## Objective
This exercise will guide students through deploying a WordPress application with a MySQL database on OpenShift. The deployment ensures WordPress runs as a non-root user and appends `-<username>` to all resource names to maintain uniqueness.

## Prerequisites
- Access to an OpenShift cluster
- Namespace: `trainingdev`
- OpenShift CLI (`oc`) installed

## Step 1: Create a MySQL Database Deployment

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret-<username> # Change This 
  namespace: trainingdev
stringData:
  MYSQL_ROOT_PASSWORD: rootpassword 
  MYSQL_DATABASE: wordpress
  MYSQL_USER: wpuser
  MYSQL_PASSWORD: wppassword
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc-<username> # Change This 
  namespace: trainingdev
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment-<username> # Change This 
  namespace: trainingdev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-<username> # Change This 
  template:
    metadata:
      labels:
        app: mysql-<username> # Change This 
    spec:
      containers:
      - name: mysql
        image: registry.redhat.io/rhel8/mysql-80:latest
        envFrom:
        - secretRef:
            name: mysql-secret-<username> # Change This 
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql/data
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc-<username> # Change This 
```

## Step 2: Create a WordPress Deployment

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-pvc-<username> # Change This 
  namespace: trainingdev
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-deployment-<username> # Change This 
  namespace: trainingdev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress-<username> # Change This 
  template:
    metadata:
      labels:
        app: wordpress-<username> # Change This 
    spec:
      securityContext:
        runAsNonRoot: true  # Ensure it runs as a non-root user
      containers:
      - name: wordpress
        image: docker.io/bitnami/wordpress:latest  # Bitnami WordPress (supports OpenShift)
        env:
        - name: WORDPRESS_DATABASE_HOST
          value: mysql-service-<username> # Change This 
        - name: WORDPRESS_DATABASE_USER
          value: wpuser
        - name: WORDPRESS_DATABASE_PASSWORD
          value: wppassword
        - name: WORDPRESS_DATABASE_NAME
          value: wordpress
        ports:
        - containerPort: 8080
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true  # Remove explicit runAsUser
        volumeMounts:
        - name: wordpress-storage
          mountPath: /bitnami/wordpress
      volumes:
      - name: wordpress-storage
        persistentVolumeClaim:
          claimName: wordpress-pvc-<username> # Change This 

```

## Step 3: Expose the Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service-<username> # Change This 
  namespace: trainingdev
spec:
  selector:
    app: mysql-<username> # Change This 
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service-<username> # Change This 
  namespace: trainingdev
spec:
  selector:
    app: wordpress-<username> # Change This 
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
---
apiVersion: route.openshift.io/v1
kind: Route
metadata: 
  name: wordpress-route-<username> # Change This 
  namespace: trainingdev
spec:
  to:
    kind: Service
    name: wordpress-service-<username> # Change This 
  port:
    targetPort: 8080
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect

```

## Deployment Instructions
1. Apply the MySQL configurations:
   ```sh
   oc apply -f mysql-deployment.yaml
   ```

2. Expose the services:
   ```sh
   oc apply -f services.yaml

3. Apply the WordPress configurations:
   ```sh
   oc apply -f wordpress-deployment.yaml
   ```
   
4. Get the WordPress route:
   ```sh
   oc get route wordpress-route-<username> -n trainingdev
   ```
5. Access WordPress via the displayed route URL.

## Conclusion
You have successfully deployed a multi-container WordPress and MySQL setup in OpenShift, ensuring unique resources for each student and running WordPress as a non-root user.
