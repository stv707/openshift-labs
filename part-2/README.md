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

#### Steps:
1. Update the deployment:
   ```sh
   oc edit deployment nodeapp-<username>
   ```
   - look for container image and change that 
   - change the image to stv707/kubia:v1

2. Verify from url that the new image is in place 

3. Rollback to a previous version:
   ```sh
   oc rollout undo deployment/nodeapp-<username>
   ```

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
