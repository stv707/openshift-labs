## Part 3: Stateful Applications & Persistent Storage

### Lab 11: Creating a Persistent Volume Claim (PVC)
#### Objective:
Request storage from the default storage class.

#### Steps:
1. Create a Persistent Volume Claim (PVC) yaml:
   - make sure you name the volume ( change the username to your name )

   ```sh
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: postgresql-pvc-<username> # Change this 
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 1Gi
   ```
   
   ```sh 
   oc apply -f volume-pvc.yaml 
   ```

2. Verify the PVC status:
   ```sh
   oc get pvc
   ```

---

### Lab 12: Deploying PostgreSQL StatefulSet
#### Objective:
Deploy a PostgreSQL instance as a StatefulSet using the PVC.

#### Steps:
1. Deploy PostgreSQL:
   - the filename is stateful.yaml
   - change the user, password and volume claim name 


   ```sh
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: postgresql-<username> # Change this 
    spec:
      serviceName: "postgresql"
      replicas: 1
      selector:
        matchLabels:
          app: postgresql-<username> # Change this
      template:
        metadata:
          labels:
            app: postgresql
        spec:
          containers:
          - name: postgresql
            image: bitnami/postgresql:latest
            env:
            - name: POSTGRESQL_USER
              value: "user"
            - name: POSTGRESQL_PASSWORD
              value: "password2025"
            - name: POSTGRESQL_DATABASE
              value: "sampledb"
            volumeMounts:
            - mountPath: "/bitnami/postgresql"
              name: postgresql-storage  # Ensure this matches the volume name below
          volumes:
          - name: postgresql-storage
            persistentVolumeClaim:
              claimName: postgresql-pvc-<username> # Change this 
   ```
   ```sh 
   oc apply -f stateful.yaml 
   ```


2. Verify the StatefulSet:
   ```sh
   oc get pods
   ```

---

### Lab 13: Connecting to PostgreSQL and Verifying Persistence
#### Objective:
Create a database, store data, and verify persistence.

#### Steps:
1. Open a shell in the PostgreSQL pod:
   ```sh
   oc rsh statefulset/postgresql-<username>
   ```
2. Connect to PostgreSQL:
   ```sh
   psql -U user -d sampledb
   ```
3. Create a table and insert data:
   ```sql
   CREATE TABLE test (id SERIAL PRIMARY KEY, name TEXT);
   INSERT INTO test (name) VALUES ('OpenShift Test');
   SELECT * FROM test;
   ```
4. Exit from the shell 
   ```sh 
   sampledb=> exit ; 
   $ exit
   ```

---

### Lab 14: Scaling the StatefulSet
#### Objective:
Scale PostgreSQL and see how persistent storage is handled.

#### Steps:
1. Scale the StatefulSet:
   ```sh
   oc scale statefulset postgresql-<username> --replicas=2
   ```
2. Verify the new pod and PVC allocation:
   ```sh
   oc get pods
   oc get pvc
   ```

3. Connect to the first instance of Database and check the data
  ```sh 
  oc rsh postgresql-<username>-0
  $ psql -U user -d sampledb -c "SELECT * FROM test;"
  $exit
  ```

4. Connect to the second instance of Database and check the data
  ```sh 
  oc rsh postgresql-<username>-1
  $ psql -U user -d sampledb -c "SELECT * FROM test;"
  $ exit
  ```
---

### Lab 15: Simulating a Pod Failure & Verifying Data Persistence
#### Objective:
Delete a PostgreSQL pod and ensure data remains intact.

#### Steps:
1. Delete the PostgreSQL pod:
   ```sh
   oc delete pod -l app=postgresql
   ```
2. Wait for the pod to restart:
   ```sh
   oc get pods -w
   ```
3. Reconnect and check if data is still there:
   ```sh
   oc rsh statefulset/postgresql-<username>
   psql -U user -d sampledb -c "SELECT * FROM test;"

   $ exit 
   ```

---
