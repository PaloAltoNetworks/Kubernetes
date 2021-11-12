# CN-Series Next-Generation Firewall Deployment

This is a repository for YAMLs to deploy [CN-Series Next-Generation Firewall](https://www.paloaltonetworks.com/network-security/cn-series) from Palo Alto Networks. To learn more about deploying CN-Series, please consult the official [Deployment Guide](https://docs.paloaltonetworks.com/cn-series/10-1/cn-series-deployment/secure-kubernetes-workloads-with-cn-series/get-the-images-and-files-for-the-cn-series-deployment.html).

## Usage

Clone the git repository to your host:
	
	git clone https://github.com/PaloAltoNetworks/Kubernetes.git
	
Change directory into the Kubernetes git repository:

	cd Kubernetes

Each YAML version is stored as part of a separate tag. To get all tags in the repository, execute the following command:

	git tag
	
Determine the YAML version number which is required for the PAN-OS version which has been deployed:

|  PAN-OS Version |  YAML Version |  CNI Version |  MGMT-INIT Version |
|---|---|---|---|
|  10.1.3 |  2.0.3 | 2.0.2  |  2.0.0 |
|  10.1.2 |  2.0.2 |  2.0.0-2.0.1 |  2.0.0 |
|  10.1.0-10.1.1 |  2.0.0-2.0.1 |  2.0.0-2.0.1 | 2.0.0  |
|  10.0.x | 1.0.x  | 1.0.x  | 1.0.0  |

Checkout the repository to the appropriate tag. For example, a PAN-OS 10.1.3 deployment requires the following checkout:

	git checkout v2.0.3 

Change directory to the appropriate sub-directory for the deployment. As an example, a Kubernetes Service deployment on GKE would require the following command:

	cd pan-cn-k8s-service/gke/
	
Edit and deploy the YAML in this directory as required.