# HCP-Gitops
This repository outlines the steps for provisioning a hosted control plane using GitOps principles.

## Prerequisites

You must meet the following prerequisites to create an OpenShift Container Platform cluster on OpenShift Virtualization:

- You need administrator access to an OpenShift Container Platform cluster, version 4.14 or later, specified by the KUBECONFIG environment variable.
- The OpenShift Container Platform hosting cluster must have wildcard DNS routes enabled, as shown in the following DNS:

      oc patch ingresscontroller -n openshift-ingress-operator default --type=json -p '[{ "op": "add", "path": "/spec/routeAdmission", 
      "value": {wildcardPolicy: "WildcardsAllowed"}}'

- The OpenShift Container Platform hosting cluster must have OpenShift Virtualization, version 4.14 or later, installed on it.
- The OpenShift Container Platform hosting cluster must be configured with OVNKubernetes as the default pod network CNI.
- The OpenShift Container Platform hosting cluster must have a default storage class. For more information, see Postinstallation storage configuration. The following example shows how to set a default storage class:
  
      oc patch storageclass ocs-storagecluster-ceph-rbd -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default- 
      class":"true"}}}'
  
- You need a valid pull secret file for the quay.io/openshift-release-dev repository.
- Before you can provision your cluster, you need to configure a load balancer. For more information, see [Configuring MetalLB](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.9/html/clusters/cluster_mce_overview#hosting-service-cluster-configure-metallb-config).

- For optimal network performance, use a network maximum transmission unit (MTU) of 9000 or greater on the OpenShift Container Platform cluster that hosts the KubeVirt virtual machines. If you use a lower MTU setting, network latency and the throughput of the hosted pods are affected. Enable multiqueue on node pools only when the MTU is 9000 or greater.
  
- The multicluster engine operator must have at least one managed OpenShift Container Platform cluster. The local-cluster is automatically imported in multicluster engine operator 2.4 and later. See Advanced configuration for more information about the local-cluster. You can check the status of your hub cluster by running the following command:
  
      oc get managedclusters local-cluster


## Create an ArgoCD Application
