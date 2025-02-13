## OpenShift Admin Section 

# OpenShift Exercise: LimitRange, ResourceQuota, Node Labeling, and Namespace Restrictions

## Objective
This exercise will guide you through:
- Creating a **project** for each student based on their email alias.
- Assigning a **user group** to the project.
- Setting up **LimitRange** and **ResourceQuota** for resource control.
- Labeling worker nodes and ensuring all workloads in the project are scheduled to specific nodes.

## Prerequisites
- An OpenShift 4.x cluster with worker nodes.
- **oc** CLI installed on your system.
- Sufficient permissions to create projects, set limits, and assign roles.

## Step 1: Define Student Projects
Each student will have a project based on their email alias:

```powershell
$students = @(
    "Khalid.ahmed", "abdelmaged", "Abdalkhalig", "Bakry", "azzaet", 
    "Sana", "ghofrana", "Linam", "ahmedya", "musabsaz", "Musabsm", "alrasheed"
)

foreach ($student in $students) {
    $namespace = $student.ToLower()
    oc new-project $namespace
    Write-Output "Project $namespace created for user $student."
}
```

## Step 2: Assign User Group to Project
Each project will be assigned an OpenShift RoleBinding to restrict access.

```powershell
foreach ($student in $students) {
    $namespace = $student.ToLower()
    
    # Assign AD user (original case) to project (lowercase)
    oc adm policy add-role-to-user edit "$student" -n "$namespace"
    
    Write-Output "Assigned AD user $student to project $namespace."
}

```

## Step 3: Set Resource Quotas for Each Project
Limit CPU and memory usage per project.

```powershell
foreach ($student in $students) {
    $namespace = $student.ToLower()

    oc apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-$namespace
  namespace: $namespace
spec:
  hard:
    pods: "15"
    requests.cpu: "2"
    requests.memory: "4Gi"
    limits.cpu: "4"
    limits.memory: "8Gi"
EOF

    Write-Output "ResourceQuota applied to project $namespace."
}

```

## Step 4: Set LimitRange for Each Project
Define container-level CPU and memory limits.

```powershell
foreach ($student in $students) {
    $namespace = $student.ToLower()

    oc apply -f - <<EOF
apiVersion: v1
kind: LimitRange
metadata:
  name: limits-$namespace
  namespace: $namespace
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
EOF

    Write-Output "LimitRange applied to project $namespace."
}

```

## Step 5: Label Worker Nodes
Label specific worker nodes for project scheduling.

```powershell
oc label node worker21 workload=student-projects
oc label node worker22 workload=student-projects
oc label node worker23 workload=student-projects
oc label node worker24 workload=student-projects
oc label node worker25 workload=student-projects
oc label node worker26 workload=student-projects
oc label node worker31 workload=student-projects
Write-Output "Worker nodes labeled for student projects."
```

## Step 6: Assign Namespace to Specific Worker Nodes
Ensure all workloads created in the project are scheduled only on worker22, worker23, worker24, and worker31 by applying a **default node selector** at the project level.

```powershell
$students = @(
    "Khalid.ahmed", "abdelmaged", "Abdalkhalig", "Bakry", "azzaet", 
    "Sana", "ghofrana", "Linam", "ahmedya", "musabsaz", "Musabsm", "alrasheed"
)

foreach ($student in $students) {
    $namespace = $student.ToLower()

    oc annotate namespace $namespace openshift.io/node-selector="workload=student-projects" --overwrite
    Write-Output "Namespace $namespace restricted to specific nodes."
}

```

## Step 7: Verify Setup
Confirm that resources are applied correctly.

```powershell
foreach ($student in $students) {
    $namespace = $student.ToLower()

    oc get project $namespace
    oc get resourcequota -n $namespace
    oc get limitrange -n $namespace
    oc get namespace $namespace --show-labels

    Write-Output "Verified project $namespace."
}

```

---

## Conclusion
By completing this exercise, you:
* ✅ Created **projects** for students.
* ✅ Assigned **user groups** and **roles**.
* ✅ Configured **ResourceQuota** and **LimitRange**.
* ✅ Labeled worker nodes and assigned namespaces to specific worker nodes.

This setup ensures proper project isolation, access control, and resource management in OpenShift. 

