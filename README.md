# CN-Series Next-Generation Firewall Deployment

This is a repository for YAMLs to deploy [CN-Series Next-Generation Firewall](https://www.paloaltonetworks.com/network-security/cn-series) from Palo Alto Networks. To learn more about deploying CN-Series, please consult the official [Deployment Guide](https://docs.paloaltonetworks.com/cn-series/10-1/cn-series-deployment/secure-kubernetes-workloads-with-cn-series/get-the-images-and-files-for-the-cn-series-deployment.html).

## Usage

Clone the git repository to your host:
	
	git clone https://github.com/PaloAltoNetworks/Kubernetes.git
	
Change directory into the Kubernetes git repository:

	cd Kubernetes

Each YAML version is stored as part of a separate tag. To get all tags in the repository, execute the following command:

	git tag
	
Determine the YAML version number which is required for the PAN-OS version which has been deployed.  Please see the CN-Series firewall image and file compatibility list here

[https://docs.paloaltonetworks.com/compatibility-matrix/cn-series-firewalls/cn-series-compatibility](https://docs.paloaltonetworks.com/compatibility-matrix/cn-series-firewalls/cn-series-compatibility)

Checkout the repository to the appropriate tag. For example, a PAN-OS 10.1.3 deployment requires the following checkout:

	git checkout v3.0.2 

Change directory to the appropriate sub-directory for the deployment. As an example, a Kubernetes Service deployment on GKE would require the following command:

	cd pan-cn-k8s-service/gke/
	
Edit and deploy the YAML in this directory as required.