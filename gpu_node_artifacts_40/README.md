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
* Priority and Preemption examples - DONE
* Taints and Toleration examples

# Examples for setting up a new machineset on AWS, in this case for nodes running CPU Manager.  CPU Manager requires two lines be added to the kubelet configuration.  In OpenShift 4 this is done with a kubeletConfig CRD.

## Create the machineset
```
oc create -f machinesets/cpumanager_machineset.yaml
```

## Add a node to the cpumanager-machineset
```
oc scale -replicas=1 machineset -n openshift-cluster-api cpumanager-machineset
```

## Create the kubeletConfig that enables CPU Manager on nodes within the cpumanager-machineset
```
oc create -f kubeletconfigs/enable-cpumanager.yaml
```

## Read back the new kubeletConfig
```
oc get kubeletconfig cpumanager-enabled -o yaml
```

The Machine Config Daemon (MCD) on the node in the cpumanager-machineset will reboot the node into the new configuration.

## Now create a pod that requests a full core
```
apiVersion: v1
kind: Pod
metadata:
  name: cpumanager-pod
spec:
  containers:
  - image: rhel7
    command: ["sleep"]
    args: ["infinity"]
    name: cpumanager-container
    resources:
      requests:
        cpu: 1
        memory: 256Mi
      limits:
        cpu: 1
        memory: 256Mi

# The cpumanager-machineset contains this label, so all nodes within that machineset are labeled automatically.
  nodeSelector:
    cpumanager: "true"
```

```
# oc create -f cpumanager-pod.yaml
