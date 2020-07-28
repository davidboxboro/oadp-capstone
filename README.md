# oadp-capstone

# Step 1: Add OADP Operator to OLM

Clone the OADP Operator repository:
```
git clone git@github.com:konveyor/oadp-operator.git
```

Create an `oadp-operator-source.yaml` file like below in oadp-operator directory:
```
apiVersion: operators.coreos.com/v1
kind: OperatorSource
metadata:
  name: oadp-operator
  namespace: openshift-marketplace
spec:
  type: appregistry
  endpoint: https://quay.io/cnr
  registryNamespace: deshah
  displayName: "OADP Operator"
  publisher: "deshah@redhat.com"
```

Remove the deployed resources:
```
oc delete -f deploy/crds/konveyor.openshift.io_v1alpha1_velero_cr.yaml
oc delete -f deploy/crds/konveyor.openshift.io_veleros_crd.yaml   
oc delete -f deploy/
oc delete namespace oadp-operator
oc delete crd $(oc get crds | grep velero.io | awk -F ' ' '{print $1}')
```

Run the following commands (note: they should be run from the root of the oadp-operator directory):

```
oc create namespace oadp-operator
oc project oadp-operator
oc create secret generic <SECRET_NAME> --namespace oadp-operator --from-file cloud=<CREDENTIALS_FILE_PATH>
oc create -f oadp-operator-source.yaml
```

# Step 2: Install OADP Operator from OperatorHub

Navigate to the OpenShift console in your web browser. Under the Administrator view, go to Operators on the left tab and click on OperatorHub. Search for the OADP Operator in the search bar. Click on it to install and subscribe to the operator. 

![OADP OperatorHub](/images/oadp_operatorhub.png)

If you go to Installed Operators and select the oadp-operator project, you should see the OADP Operator successfully installed:

![OADP Installed](/images/oadp_installed.png)

In order to use OLM for OADP deployment , you need to change flag `olm_managed` in the `konveyor.openshift.io_v1alpha1_velero_cr.yaml` to true. The file is present in deploy/crds folder.

For instance the `konveyor.openshift.io_v1alpha1_velero_cr.yaml` file might look something like this:

```
apiVersion: konveyor.openshift.io/v1alpha1
kind: Velero
metadata:
  name: example-velero
spec:
  use_upstream_images: true
  olm_managed: true
  default_velero_plugins:
  - aws
```

When the installation succeeds, create a Velero CR
```
oc create -f deploy/crds/konveyor.openshift.io_v1alpha1_velero_cr.yaml
```

Post completion of all the above steps, you can check if the operator was successfully installed; the expected result for the command `oc get all -n oadp-operator` is as follows:
```
NAME                                             READY   STATUS    RESTARTS   AGE
pod/oadp-default-aws-registry-568978c9dc-glpfj   1/1     Running   0          10h
pod/oadp-operator-64f79d9bf4-4lzl9               1/1     Running   0          10h
pod/restic-bc5tm                                 1/1     Running   0          10h
pod/restic-dzrkh                                 1/1     Running   0          10h
pod/restic-z4mhx                                 1/1     Running   0          10h
pod/velero-779f785b7d-5z6qf                      1/1     Running   0          10h

NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/oadp-default-aws-registry-svc   ClusterIP   172.30.155.164   <none>        5000/TCP            10h
service/oadp-operator-metrics           ClusterIP   172.30.58.121    <none>        8383/TCP,8686/TCP   10h

NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/restic   3         3         3       3            3           <none>          10h

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oadp-default-aws-registry   1/1     1            1           10h
deployment.apps/oadp-operator               1/1     1            1           10h
deployment.apps/velero                      1/1     1            1           10h

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/oadp-default-aws-registry-568978c9dc   1         1         1       10h
replicaset.apps/oadp-operator-64f79d9bf4               1         1         1       10h
replicaset.apps/velero-779f785b7d                      1         1         1       10h

NAME                                                       HOST/PORT                                                                                        PATH   SERVICES                        PORT       TERMINATION   WILDCARD
route.route.openshift.io/oadp-default-aws-registry-route   oadp-default-aws-registry-route-oadp-operator.apps.cluster-dshah-4-5.dshah-4-5.mg.dog8code.com          oadp-default-aws-registry-svc   5000-tcp                 None
``` 

# Step 3: Install OCS from OperatorHub

Navigate to the OpenShift console. Under the Administrator view, go to Operators on the left tab and click on OperatorHub. Search for the OpenShift Container Storage operator in the search bar. Click on it to install and subscribe to the operator. 

![OCS OperatorHub](/images/ocs_operatorhub.png)

If you go to Installed Operators and select the openshift-storage project, you should see the OpenShift Container Storage operator successfully installed:

![OCS Installed](/images/ocs_installed.png)
