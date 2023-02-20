[![Repository on Quay](https://quay.io/repository/rimitche/packet-capture-operator/status "Repository on Quay")](https://quay.io/repository/rimitche/packet-capture-operator)

# Caution
Note that this is a Work In Progress and not a complete solution.  The current capabilities does the following:
- Create the PacketCapture operator and PacketCaptures CRD
- When a PacketCapture CR is created the operator creates a ConfigMap and a job per Pod found for the Namespace / Deployment
- Cleans up when the PacketCapture CR is deleted

The properties of the CRD and the objects that get created are likely to change as the solution develops.

# packet-capture-operator
Operator SDK based ansible operator for performing packet capture using tcpdump to generate PCAP files.  When the operator is deployed it creates the PacketCapture CRD and starts managing the status of any PacketCapture CRs created form that CRD.  

The PacketCapture CRD defines CRs with the following YAML structure:
```yaml
apiVersion: monitoring.openshift.tpg/v1alpha1
kind: PacketCapture
metadata:
  labels:
    app.kubernetes.io/name: packet-capture
    app.kubernetes.io/instance: packet-capture-{{targetNamespace}}-{{targetDeployment}}
    app.kubernetes.io/part-of: packet-capture-operator
    app.kubernetes.io/managed-by: kustomize
    app.kubernetes.io/created-by: packet-capture-operator
  name: packet-capture-{{targetNamespace}}-{{targetDeployment}}
  namespace: packet-capture
spec:
  jobId: {{jobId}}
  targetNamespace: {{targetNamespace}}
  targetDeployment: {{targetDeployment}}
  duration: {{duration}}
```
Example
```yaml
apiVersion: monitoring.openshift.tpg/v1alpha1
kind: PacketCapture
metadata:
  labels:
    app.kubernetes.io/name: packet-capture
    app.kubernetes.io/instance: packet-capture-openshift-gitops-openshift-gitops-server
    app.kubernetes.io/part-of: packet-capture-operator
    app.kubernetes.io/managed-by: kustomize
    app.kubernetes.io/created-by: packet-capture-operator
  name: packet-capture-openshift-gitops-openshift-gitops-server
  namespace: packet-capture
spec:
  jobId: qwer
  targetNamespace: openshift-gitops
  targetDeployment: openshift-gitops-server
  duration: "30"
```

The labels are key to the management of the objects that get created and the `app.kubernetes.io/instance` label value will be replicated through each of the created objects.  When the CR is later deleted, all objects that have been created with that label will also be deleted as part of the clean up.
| Property | Type | Description |
| --- | --- | --- |
| spec.jobId | String | 4 character string that is used to make the Job names umnique if the same target Pods are monitored on more that one occasion.
| spec.targetNamespace | String | The Namespace where the target Pods are running
| spec.targetDeployment | String | The Deployment that manages the Pods
| spec.duration | String | The number of seconds for the monitoring to run

When a PacketCapture CR is created the operator will call the ansible role and that will create the following:

| Kind | Name | Quantity | Description |
| --- | --- | :---: | --- |
| ConfigMap | {{targetNamespace}}-{{targetDeployment}} | 1 | The ConfigMap for this particular packet capture execution
| Job | {{podName}}-{{jobId}} | One per Pod | A Job that will start the privileged container to perform the tcpdump
