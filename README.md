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


## Configure ArgoCD
### Prerequisites
Prerequisites
- A running Argo CD deployment
- Access to the Git repository containing your application manifests (HTTPS URL or SSH details)
- Authentication credentials for the Git repository (if required)

### Adding a Repository using Argo CD UI
1. Access Argo CD UI: Open your Argo CD dashboard in your web browser.

2. Navigate to Settings: Click on the Settings option in the left-hand navigation menu.

3. Manage Repositories: Under Settings, select Repositories.

4. Connect a New Repository: Click the Connect Repo Using... button. Choose the appropriate connection method based on your Git repository (SSH, HTTPS, or GitHub App).

5. Fill in Repository Details:
   - Type: Select git.

   - Name: Provide a unique name to identify this repository within Argo CD.

   - Repository URL: Enter the URL of your Git repository (e.g., HTTPS URL or SSH URI).

   - Credentials (if private): For private repositories, enter the username and password (HTTPS) or configure SSH key access.

   - Username: Username for your Git repository account (if required).
   - Password: Password for your Git repository account (if required).
   - SSH Private Key: You can upload your private key file directly in the UI (not recommended for production due to security concerns).
6. Save the Configuration: Click the Create button to establish the connection with your Git repository.


### Creating an Argo CD Application

1. Access Argo CD UI: Open your Argo CD dashboard in your web browser.

2. Navigate to Applications: Click on the Applications tab in the left-hand navigation menu.

3. Create a New Application: Click the Create Application button.

4. Application Details:
      - Name: Provide a unique name for your application.
      - Project: Select the project where you want to deploy the application (optional, can be created during application creation).
      - Sync Policy: Choose the desired synchronization behavior (Automated, Manual, or Hook).

        1. Automated: Argo CD automatically syncs and deploys changes from the Git repository.
                        -  Manual: You initiate deployments by manually syncing the application.
            3. Hook: External triggers like CI/CD pipelines can initiate syncing.
5. Source Details:
   - Git Repository: Select the previously connected Git repository from the dropdown menu.
   - Path: Specify the path within your Git repository that points to the deployment manifest file(s) for this application. In our case it is "."

6. Destination Details:
      - Cluster: Choose the Kubernetes cluster where you want to deploy the application resources.

7. Leave the other fields blank and click the Create button to create the Argo CD application.

## Helm Chart Structure:
The Helm chart follows the standard structure:
   - Chart.yaml: Contains the HCP chart definition files.
   - templates: Contains the template files used for generating deployment manifests.
   - values.yaml: The default configuration values for the HCP cluster.

## Using the Helm Chart
1. Review values.yaml:
The values.yaml file defines the default configuration for your HCP cluster. Carefully review the options and modify them according to your specific requirements. This file controls aspects like worker_count, CPU, Memory, and more.

2. To customize the HCP cluster provisioning process, modify the following:
   - values.yaml: This is the primary file for overriding default settings. Adjust any configuration options as needed.
   - templates: For advanced customization, you can modify the template files within the templates directory.
