# Installing Tanzu Application Platform (offline)

To install Tanzu Application Platform on your Kubernetes clusters in an air-gapped environment:

|Step|Task|Link|
|----|----|----|
|1.| Review the prerequisites to ensure you have met all requirements before installing. |[Prerequisites](prerequisites.hbs.md)|
|2.| Accept Tanzu Application Platform EULAs and install the Tanzu CLI. |[Accepting Tanzu Application Platform EULAs and installing the Tanzu CLI](install-tanzu-cli.hbs.md)|
|3.| Install Cluster Essentials for Tanzu*. |[Deploying Cluster Essentials](https://docs.vmware.com/en/Cluster-Essentials-for-VMware-Tanzu/{{ vars.url_version }}/cluster-essentials/GUID-deploy.html)|
|4.| Add the Tanzu Application Platform package repository, prepare your Tanzu Application Platform profile, and install the profile to the cluster. |[Installing Tanzu Application Platform in an air-gapped environment](install-air-gap.hbs.md)|
|5.| Install Tanzu Build Service full dependencies. |[Installing the Tanzu Build Service dependencies](tbs-offline-install-deps.hbs.md)|
|6.| Configure custom certificate authorities for Tanzu Application Platform GUI. |[Configuring custom certificate authorities for Tanzu Application Platform GUI](tap-gui-non-standard-certs-offline.hbs.md) |
|7.| Add the certificate for the private Git repository in the Accelerator system namespace. |[Configuring Application Accelerator](application-accelerator-configuration-offline.hbs.md)|
|8.| Apply patch to Grype. |[Using Grype in offline and air-gapped environments](grype-offline-airgap.hbs.md)|
|9.| Set up developer namespaces to use installed packages. |[Setting up developer namespaces to use installed packages](set-up-namespaces-offline.hbs.md)|

\* _When you use a VMware Tanzu Kubernetes Grid cluster, there is no need to install Cluster Essentials because the contents of Cluster Essentials are already installed on your cluster._

After installing Tanzu Application Platform on to your air-gapped cluster, you can start creating workloads that run in your air-gapped containers.
For instructions, see [Deploy your first air-gapped workload](getting-started/air-gap-workload.hbs.md).