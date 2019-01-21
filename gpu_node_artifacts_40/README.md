# Examples for setting up a new machineset on AWS, in this case for GPUs, scaling that machineset, and debugging it.
```
oc create  -f gpu_machineset.yaml --validate=true --dry-run
oc create  -f gpu_machineset.yaml

oc get machinesets -n openshift-cluster-api gpu-machineset -o yaml

oc scale --replicas=1 machineset -n openshift-cluster-api gpu-machineset
oc scale --replicas=2 machineset -n openshift-cluster-api gpu-machineset
oc scale --replicas=0 machineset -n openshift-cluster-api gpu-machineset

oc logs -n openshift-cluster-api $(oc get pods -n openshift-cluster-api|grep clusterapi-manager-controllers-|awk '{print $1}') -c machine-controller -f
```

# TODO:
* Node Feature Discovery deployment
* Usage of GPU Operator
* Basic GPU cuda-vector-add example
* Priority and Preemption examples
* Taints and Toleration examples
