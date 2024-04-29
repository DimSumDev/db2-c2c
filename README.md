## [db2-c2c](https://www.ibm.com/docs/en/db2/11.5?topic=deployments-click-containerize)
#### This repo used to install Db2 on K8s of your choice. Db2 container requrires the Db2 operator.
#### Namespace
#### storageclass
#### 

## Prerequisites 
### 1)  [Operator Lifecycle Manager (OLM)](https://sdk.operatorframework.io/docs/installation/)
1a) Install the Operator SDK CLI using GitHub release  
Set platform information:
```
export ARCH=$(case $(uname -m) in x86_64) echo -n amd64 ;; aarch64) echo -n arm64 ;; *) echo -n $(uname -m) ;; esac)
export OS=$(uname | awk '{print tolower($0)}')
```
Download the binary for your platform:
```
export OPERATOR_SDK_DL_URL=https://github.com/operator-framework/operator-sdk/releases/download/v1.34.1
curl -LO ${OPERATOR_SDK_DL_URL}/operator-sdk_${OS}_${ARCH}
```
> [!Tip]
> [To verify the downloaded binaries](https://sdk.operatorframework.io/docs/installation/#2-verify-the-downloaded-binary)

Install the release binary in your PATH
```
chmod +x operator-sdk_${OS}_${ARCH} && sudo mv operator-sdk_${OS}_${ARCH} /usr/local/bin/operator-sdk
```
1b) [Install OLM](https://olm.operatorframework.io/docs/getting-started/#installing-olm-in-your-cluster)
```
operator-sdk olm install
```
### 2) Install Db2U Operator
2a) Create file `db2uoperator.yaml` with following information
```
apiVersion: v1
kapiVersion: v1
kind: Namespace
metadata:
  name: db2u
---
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: db2u-catalog
  namespace: olm
spec:
  sourceType: grpc
  image: icr.io/db2u/db2-catalog@sha256:e6ac7746ae2d6d3bb4fbf75144d94e400db887c1702b19e08b9533752d896178
  displayName: Db2u Operators
  publisher: IBM Db2
---
apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  name: db2u-oprator-og
  namespace: db2u
spec:
  targetNamespaces:
  - db2u
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: db2u-operator-v2-0-0-sub
  namespace: db2u
spec:
  name: db2u-operator
  source: db2u-catalog
  sourceNamespace: olm
  installPlanApproval: Automatic
```
> [!NOTE]
> To find latest version of Db2U operator and Db2, goto [IBM Documentation](https://www.ibm.com/docs/en/db2/11.5?topic=deployments-db2-rhos-k8s). Expand Left side menu and pick version.  
> Db2 for RHOS and K8s -> \<pick version\> -> Installing Db2 -> Installing the Db2 Operator -> Installing OLM to deploy the Db2 Operator  
> If you are deploying Db2 in multiple namespaces use `namespace-scope`
### 3) Create Db2 Operator 
```
kubectl create -f \<file path\>/db2uoperator.yaml
```
### 4) Deploy Db2 in `db2u` namespace
Create file `db2create.yaml` with following information
```
apiVersion: db2u.databases.ibm.com/v1
kind: Db2uCluster
metadata:
  name: demo 
  namespace: db2u
spec:
  account:
    privileged: true
  environment:
    dbType: db2oltp
    instance:
      password: db2oltp
    ldap:
      enabled: false
    database:
      name: DB2OLTP
  license:
    accept: true
  podConfig:
    db2u:
      resource:
        db2u:
          limits:
            cpu: '4'
            memory: 8Gi
          requests:
            cpu: '4'
            memory: 8Gi
  size: 1
  version: "11.5.7.0"
  storage:
    - name: meta
      type: "create"
      spec:
        storageClassName: <createdstorageclass>
        accessModes:
          - ReadWriteMany
        resources:
          requests:
            storage: 5Gi
    - name: data
      type: "create"
      spec:
        storageClassName: <createdstorageclass>
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 5Gi
```
> [!NOTE]
> For information on `spec:` confiuration options goto [IBM Documentation: Db2uInstance custom resource](https://www.ibm.com/docs/en/db2/11.5?topic=resource-deploying-db2-using-db2uinstance-custom)  

### Apply the `yaml`
```
kubectl create -f \<file path\>/db2create.yaml
```
> [!NOTE]
> It can take several minutes for all the pods to come up.
> Use command to watch pods come up. Example: `kubectl get pods -n db2u -w`
```
db2u-operator-manager-c849d76c4-xl9bs   1/1     Running
c-demo-instdb-qc42j                     0/1     Completed
c-demo-etcd-0                           1/1     Running
c-demo-db2u-0                           1/1     Running
c-demo-restore-morph-ghvln              0/1     Completed
```
