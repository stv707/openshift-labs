## Part 2: Advanced OpenShift Operations

### Lab 6: Deploying from GitHub Repository
#### Objective:
Deploy a containerized application directly from a GitHub repository.

#### Steps:
1. Use the **Node.js Example Repository**:
   ```sh
   oc new-app https://github.com/sclorg/nodejs-ex.git --name=nodeapp-<username>
   ```
2. Verify that the build process has started:
   ```sh
   oc logs -f bc/nodeapp-<username>
   ```
3. Check the pods:
   ```sh
   oc get pods
   ```

---

### Lab 7: Exposing and Verifying Deployment
#### Objective:
Expose the deployed GitHub application and verify its accessibility.

#### Steps:
1. Expose the service:
   ```sh
   oc expose svc/nodeapp-<username>
   ```
2. Get the route and verify:
   ```sh
   oc get routes
   ```
3. Open the application URL in a browser.

---

### Lab 8: Managing Build and Deployments
#### Objective:
Understand OpenShift's automated builds and deployments.

#### Steps:
1. Trigger a manual build:
   ```sh
   oc start-build nodeapp-<username>
   ```
2. View deployment configurations:
   ```sh
   oc get deployment
   ```

---

### Lab 9: Rolling Updates and Rollbacks
#### Objective:
Perform rolling updates and rollbacks in OpenShift.
### Updated steps to test with a ready made Container Image: 

1. Run this to create a new App: 

   ```sh 
   oc new-app --name=kubia-<username> --image=stv707/kubia:v1 
   ```

2. Expose the application
   ```sh 
   oc expose deployment kubia-<username> --port=8080 --target-port=8080

   oc expose service/kubia-<username>
   ```

3. Get the route and verify the application is working 
   ```sh 
   oc get routes 
   ```

4. Update the image in the deployment
   ```sh 
   oc set image deployment/kubia-<username> kubia-<username>e=stv707/kubia:v2
   ```
   - Verify the application showing v2 now 

5. Rollback to a previous version:
   ```sh
   oc rollout undo deployment/kubia-<username>
   ```
   -- Verify the application showing v1 now ( rollback )
---

### Lab 10: Resource Limits and Auto-Scaling
#### Objective:
Set resource limits and configure auto-scaling for the deployment.

#### Steps:
1. Apply resource limits:
   ```sh
   oc set resources deployment nodeapp-<username> --limits=cpu=500m,memory=256Mi
   ```
2. Enable auto-scaling:
   ```sh
   oc autoscale deployment nodeapp-<username> --min=1 --max=5 --cpu-percent=80
   ```
---

## Notes
- Ensure all resource names include `-<username>` to prevent conflicts in the shared namespace.
- Use `oc delete` if you need to clean up resources after each lab:
  ```sh
  oc delete all -l app=nodeapp-<username>
  ```
