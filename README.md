# OpenShift-AI-Toolkit-Operator
This operator provides Deployment Server for various models available on IBM Z

## Operator Based on
[IBM Z Nvidia triton Repo](https://github.com/IBM/ibmz-accelerated-for-nvidia-triton-inference-server?tab=readme-ov-file#models-repository)

## PREREQUISITES
- IBM Cloud Account
- Openshift Cluster on IBM Z
- Openshift Client
 
## Installation

### Step 1: Setup ICR Auth credientials in openshift-config

- Define Enviroment

  ```sh
  (base) shivanggoswami@Shivangs-MacBook-Pro operator-setup % cat env-template.sh 
  export REGISTRY_URL="icr.io" #The Registry URL. Default: icr.io
  export EMAIL="person@ibm.com" #IBM Cloud Userid. Format: user@ibm.com
  export API_KEY="api-key" #IBM Cloud API key
  ```

  Modify the Following file and load the environment variables in the system

  After this, First login into your openshift cluster and execute this script

  ```sh
  (base) shivanggoswami@Shivangs-MacBook-Pro operator-setup % source env.sh # this is derived from env-templates 
  (base) shivanggoswami@Shivangs-MacBook-Pro operator-setup % ./populateICRconfig.sh 
  secret/icr-registry created
  secret "icr-registry" deleted
  secret/pull-secret data updated
  ```

  The following command can be used to assert the script worked properly
  ```sh
  (base) shivanggoswami@Shivangs-MacBook-Pro operator-setup % oc get secret pull-secret -n openshift-config -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d
  ...
   "icr.io": {
      "username": "iamapikey",
      "password": <ibm-api-key>,
      "email": <ibm-user-id>,
      "auth": <username/password in base64 encoding>
    }
   }
  }
  ```
### Step 2: Apply Catalog Source

- Apply Catalog Source
  ```sh
  (base) shivanggoswami@Shivangs-MacBook-Pro operator-setup % oc apply -f catalog-source.yaml
  catalogsource.operators.coreos.com/openshift-ai-toolkit-operator-catalog created
  ```

  If everything goes through fine, operator will be available in operator hub and can be installed from there
  ![Screenshot 2024-12-05 at 9 36 25 PM](https://github.ibm.com/Openshift-ai-toolkit/OpenShift-AI-Toolkit-Operator/assets/407662/add5d509-cd70-41a6-aad5-f2b77774dab8)

## Using this Operator

### Step 1: Creating pvc and injecting Model data inside them

As an example, there is a model directory present in the repo with two pre-populated data modules: creadit card fraud detection(Snapml) and densenet(onnx)
```sh
(base) shivanggoswami@Shivangs-MacBook-Pro operator-examples % tree models         
models
├── cc_fraud_detect_model
│   ├── 1
│   │   └── model.pmml
│   └── config.pbtxt
└── densenet_onnx
    ├── 1
    │   └── model.so
    └── config.pbtxt

5 directories, 4 files
```
This Directory structure and model formats can be referred from the parent repo.

First create a openshift namespace, then populate data values in this script
```sh
(base) shivanggoswami@Shivangs-MacBook-Pro operator-examples % cat create-sync-pvc.sh 
...
# Set your variables
LOCAL_DIR="./models"    # Local directory to copy data from
NAMESPACE="test"        # Kubernetes namespace
PVC_NAME="triton-pvc"   # PVC name
POD_NAME="alpine-pvc-pod" # Pod name
MOUNT_PATH="/mnt/data"  # PVC mount path inside the pod
CONTAINER_NAME="alpine" # Container name
PVC_STORAGE="10Gi"      # PVC storage size
ALPINE_IMAGE="alpinelinux/rsyncd" # Alpine image to use for the pod
CLEAN=true             # Set to true to clean (delete) existing PVC and Pod before execution
...
```

The following script use oc rsync command so local directory can sync delta changes with pvc if the pvc already exists
| Variable Name   | Sample Value           | Explanation                                                   |
|------------------|------------------------|---------------------------------------------------------------|
| `LOCAL_DIR`     | `./models`            | Local directory path containing the data to copy.             |
| `NAMESPACE`     | `test`                | The Kubernetes namespace to operate in.                      |
| `PVC_NAME`      | `triton-pvc`          | Name of the Persistent Volume Claim to create or use.         |
| `POD_NAME`      | `alpine-pvc-pod`      | Name of the pod to be created or managed.                     |
| `MOUNT_PATH`    | `/mnt/data`           | Path where the PVC will be mounted inside the pod.            |
| `CONTAINER_NAME`| `alpine`              | Name of the container within the pod.                         |
| `PVC_STORAGE`   | `10Gi`                | Storage size requested for the PVC.                           |
| `ALPINE_IMAGE`  | `alpinelinux/rsyncd`  | Docker image to use for the pod's container.                  |
| `CLEAN`         | `true`                | Indicates whether to delete existing PVC and pod before execution. |

Attaching sample Output for Reference:
<details> <summary>Click to expand: Shell Output</summary>
 
 ```sh
 [root@t313lp68 operator-examples]# ./create-sync-pvc.sh 
 Creating PVC triton-pvc...
 persistentvolumeclaim/triton-pvc created
 PVC is already bound, no need to wait.
 Creating pod alpine-pvc-pod with Alpine image and PVC mount...
 Warning: would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (container "alpine" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "alpine" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "alpine" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "alpine" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
 pod/alpine-pvc-pod created
 Waiting for pod alpine-pvc-pod to start...
 Waiting for pod alpine-pvc-pod to be ready...
 pod/alpine-pvc-pod condition met
 Copying data from ./models to /mnt/data inside pod alpine-pvc-pod...
 sending incremental file list
 ./
 cc_fraud_detect_model/
 cc_fraud_detect_model/config.pbtxt
             459 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=5/9)
 cc_fraud_detect_model/1/
 cc_fraud_detect_model/1/model.pmml
       2,375,804 100%   75.52MB/s    0:00:00 (xfr#2, to-chk=3/9)
 densenet_onnx/
 densenet_onnx/config.pbtxt
             246 100%    8.01kB/s    0:00:00 (xfr#3, to-chk=2/9)
 densenet_onnx/1/
 densenet_onnx/1/model.so
      33,201,936 100%   19.65MB/s    0:00:01 (xfr#4, to-chk=0/9)
 
 sent 30,371,036 bytes  received 127 bytes  20,247,442.00 bytes/sec
 total size is 35,578,445  speedup is 1.17
 Data successfully copied to PVC.
 Script execution completed in 6 seconds.
 ```
</details>


