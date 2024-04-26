## [db2-click](https://www.ibm.com/docs/en/db2/11.5?topic=deployments-click-containerize)
##### This repo used to install Db2 on K8s of your choice. Db2 container requrires the Db2 operator

### Prerequisites 
#### 1)  [Operator Lifecycle Manager (OLM)](https://sdk.operatorframework.io/docs/installation/)
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
#### 2) Install Db2U Operator
2a) Create `db2uoperator.yaml` with following information
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
> Db2 for RHOS and K8s -> {pick version} -> Installing Db2 -> Installing the Db2 Operator -> Installing OLM to deploy the Db2 Operator
> If you are deploying Db2 in multiple namespaces use `namespace-scope`


https://www.ibm.com/docs/en/db2/11.5?topic=deployments-db2-rhos-k8s

