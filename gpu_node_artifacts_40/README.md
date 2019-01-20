oc create  -f gpu_machineset.yaml --validate=true
oc get machinesets -n openshift-cluster-api gpu-machineset -o yaml
oc scale --replicas=1 machineset -n openshift-cluster-api gpu-machineset
oc scale --replicas=2 machineset -n openshift-cluster-api gpu-machineset
oc scale --replicas=0 machineset -n openshift-cluster-api gpu-machineset
oc logs -n openshift-cluster-api $(oc get pods -n openshift-cluster-api|grep clusterapi-manager-controllers-|awk '{print $1}') -c machine-controller -f
