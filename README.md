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





