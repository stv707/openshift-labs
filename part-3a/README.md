# Deploying Microsoft SQL Server Container on OpenShift

## Overview
In this exercise, you will deploy Microsoft SQL Server as a container on OpenShift. Each student must modify the resources to include their unique username (e.g., `mssql-user123`) to prevent conflicts when deploying in the same namespace.

## Prerequisites
- Access to an OpenShift cluster
- `oc` CLI installed and logged in
- Storage configured for Persistent Volumes


Replace `<username>` with your actual username.

## Step 1: Deploy Microsoft SQL Server Data Volume 
Modify this file ( mssql-data.yaml ) to set your username.
```sh
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mssql-data-<username> 
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```

- Run the command to create PVC and verify its ready 

```sh 
oc apply -f mssql-data.yaml

oc get pvc 
```



## Step 2: Create Secret to Hold MSSQL Password
- Ensure you replace `<username>` with your OpenShift username.

# If you change the password!!! USE Microsoft Standard Password Policy for Microsoft SQL !!!!
```sh
oc create secret generic mssql-<username> --from-literal=MSSQL_SA_PASSWORD="MyStrong!Passw0rd"
```

- Verify the secret is created:

```sh
oc get secret mssql-<username> 
```

## Step 3: Create MSSQL Deployment.
- Create a file called mssql-deployment.yaml or modify the existing file within this directory
- To ensure data persistence, change all the  `mssql-<username>` to your username

```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mssql-deployment-<username> #Change this 
spec:
  replicas: 1
  selector:
     matchLabels:
       app: mssql
  template:
    metadata:
      labels:
        app: mssql
    spec:
      terminationGracePeriodSeconds: 30
      hostname: mssqlinst
      securityContext:
        fsGroupChangePolicy: OnRootMismatch
      containers:
      - name: mssql
        image: mcr.microsoft.com/mssql/server:2022-latest
        resources:
          requests:
            memory: "1G"
            cpu: "1000m"
          limits:
            memory: "2G"
            cpu: "2000m"
        ports:
        - containerPort: 1433
        securityContext:
          capabilities:
            add:
            - NET_BIND_SERVICE
        env:
        - name: MSSQL_PID
          value: "Developer"
        - name: ACCEPT_EULA
          value: "Y"
        - name: MSSQL_TCP_PORT
          value: "1433" 
        - name: MSSQL_SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mssql-<username> #Change this 
              key: MSSQL_SA_PASSWORD
        volumeMounts:
        - name: mssqldb
          mountPath: /var/opt/mssql
      volumes:
      - name: mssqldb
        persistentVolumeClaim:
          claimName: mssql-data-<username> #Change this 
---
apiVersion: v1
kind: Service
metadata:
  name: mssql-deployment-<username> #Change this 
spec:
  selector:
    app: mssql
  ports:
    - protocol: TCP
      port: 1433
      targetPort: 1433
  type: ClusterIP

```
- Deploy the file mssql-deployment.yaml
- verify the deployment of mssql 


```sh 
oc apply -f mssql-deployment.yaml

oc get pod 

oc get deployment 

oc get pvc 

oc get service 

```


## Step 4: Verify Deployment logs 
To check logs:
```sh
oc logs deployment/mssql-deployment-<username>
```

## Step 5: Accessing SQL Server
To connect to the SQL Server container, open a terminal session inside the running pod:
```sh
oc rsh deployment/mssql-deployment-<username> bash
```
Inside the pod, connect using `sqlcmd`:
```sh
/opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'MyStrong!Passw0rd' -C
```

## Step 6: Create a Sample Database
Run the following commands inside `sqlcmd`:
```sql
CREATE DATABASE SampleDB;
GO
```

```sql 
USE SampleDB;
GO
```

```sql 
CREATE TABLE Customers (ID INT PRIMARY KEY, Name NVARCHAR(50));
GO
```

```sql 
INSERT INTO Customers VALUES (1, 'John Doe');
GO
```

```sql 
SELECT * FROM Customers;
GO
```


To exit `sqlcmd`, type:

```sql
EXIT
```

Exit the continer: 
```sh 
exit 
```

## Step 7: Delete the deployment and recreate : 

- in this step you will delete the deployment and recreate to ensure the data is not lost as we are using PVC 

- Delete the deployment and verify the container MSSQL is not there
```sh
oc delete -f .\mssql-deployment-sol.yaml

oc get pod

oc get pvc

```


- Recreate the deployment: 
```sh
oc apply -f mssql-deployment.yaml

oc get pod 

oc get deployment 

oc get pvc 

oc get service 
```

- Verify the database and data are intact: 

```sh
oc rsh deployment/mssql-deployment-<username> bash
```
Inside the pod, connect using `sqlcmd`:
```sh
/opt/mssql-tools18/bin/sqlcmd -S localhost -U SA -P 'MyStrong!Passw0rd' -C
```

```sql 
USE SampleDB;
GO
```

```sql 
SELECT * FROM Customers;
GO
```

To exit `sqlcmd`, type:

```sql
EXIT
```

Exit the continer: 
```sh 
exit 
```

