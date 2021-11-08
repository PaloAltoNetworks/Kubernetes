# CN-Series Next-Generation Firewall Deployment

This is a repository for YAMLs to deploy CN-Series Next-Generation firewall from Palo Alto Networks.

**Supported modes for Next-Generation Firewall Deployment**

2 modes of deployment are supported:
1. pan-cn-k8s-daemonset - Supported starting PANOS 10.0.0 release. The NGFW pods run in daemon set mode.
2. pan-cn-k8s-service - Supported starting PANOS 10.1.0 release. NGFW kubernetes service secures pods with NGFW pods as endpoints.

All the YAMLs required to deploy CN-Series on a given cloud platform are present under that cloud platform specific directory for each deployment mode. Users can use these YAMLs as is to deploy CN-Series quickly after filling in just these fields from their setup:
  ```
  In pan-cni.yaml, pan-cn-mgmt.yaml and pan-cn-ngfw.yaml:
      image: <your-private-registry-image-path>

  In pan-cn-mgmt-secret.yaml:
      PAN_PANORAMA_AUTH_KEY: <panorama-auth-key>
      # Thermite Certificate retrieval 
      CN-SERIES-AUTO-REGISTRATION-PIN-ID: "<PIN Id>"
      CN-SERIES-AUTO-REGISTRATION-PIN-VALUE: "<PIN-Value>"

  In pan-cn-mgmt-configmap.yaml:
      # Panorama settings
      PAN_PANORAMA_IP: <panorama-IP>
      PAN_DEVICE_GROUP: <panorama-device-group>
      PAN_TEMPLATE_STACK: <panorama-template-stack>
      PAN_CGNAME: <panorama-collector-group>
  ```
For production deployment, it's expected users would want to customize the YAMLs as per below:
  - Resources (cpu, memory) fields in pan-cn-mgmt.yaml and pan-cn-ngfw.yaml are pre-populated but should be customized to better suit the deployment scenario.
  - There are some optional fields in the configmaps which users can add e.g. PAN_PANORAMA_IP2 for Panorama in HA, or CLUSTER_NAME for easier identification when managing multiple Kubernetes clusters under the same Panorama.
  **Note**: For complex setup and advanced topics needing modifications in the YAMLs, refer to the deployment documentations for details. Changing a field might require modification in multiple places and multiple YAMLs.


Once the YAMLs have been modified as desired, these YAMLs should be deployed in the order preferably.

**YAMLs for pan-cn-k8s-daemonset mode**
Please ensure that pan-cn-mgmt-slot-crd.yaml is deployed before pan-cn-mgmt-slot-cr.yaml:
```
kubectl apply -f plugin-serviceaccount.yaml
kubectl apply -f pan-cni-serviceaccount.yaml
kubectl apply -f pan-mgmt-serviceaccount.yaml
kubectl apply -f pan-cni-configmap.yaml
kubectl apply -f pan-cni.yaml
kubectl apply -f pan-cn-mgmt-secret.yaml
kubectl apply -f pan-cn-mgmt-configmap.yaml
kubectl apply -f pan-cn-mgmt-slot-crd.yaml
sleep 1 # This ensures the CRD is created before we create a resource of that kind
kubectl apply -f pan-cn-mgmt-slot-cr.yaml
kubectl apply -f pan-cn-mgmt.yaml
kubectl apply -f pan-cn-ngfw-configmap.yaml
kubectl apply -f pan-cn-ngfw.yaml
```

**YAMLs for pan-cn-k8s-service mode**
Please ensure that pan-cn-ngfw-svc.yaml is deployed before pan-cni.yaml and pan-cn-mgmt-slot-crd.yaml is deployed before pan-cn-mgmt-slot-cr.yaml:
```
kubectl apply -f plugin-serviceaccount.yaml
kubectl apply -f pan-cni-serviceaccount.yaml
kubectl apply -f pan-mgmt-serviceaccount.yaml
kubectl apply -f pan-cni-configmap.yaml
kubectl apply -f pan-cn-ngfw-svc.yaml
kubectl apply -f pan-cni.yaml
kubectl apply -f pan-cn-mgmt-secret.yaml
kubectl apply -f pan-cn-mgmt-configmap.yaml
kubectl apply -f pan-cn-mgmt-slot-crd.yaml
sleep 1 # This ensures the CRD is created before we create a resource of that kind
kubectl apply -f pan-cn-mgmt-slot-cr.yaml
kubectl apply -f pan-cn-mgmt.yaml
kubectl apply -f pan-cn-ngfw-configmap.yaml
kubectl apply -f pan-cn-ngfw.yaml
```

To enable the security for the application pods, apply the following annotation to their YAMLs, OR, to enable the security for all the pods in a given namespace, apply this annotation to the namespace: ```     paloaltonetworks.com/firewall: pan-fw``` e.g. for "default" namespace 
```
kubectl annotate namespace default paloaltonetworks.com/firewall=pan-fw
```

**Horizontal Pod Scaling in k8s-service mode**

For HPA in GKE environment, it's expected users would want to customize the YAMLs as per below:
  -  Update the metrics that would be used for scaling dps in pan-cn-hpa-dp.yaml and mps pan-cn-hpa-mp.yaml.
  -  If mp is deployed in custom namespace, update the pan-cn-adapter yaml with custom namespace instead of kubesystem.
  -  Update the pan-cn-hpa-dp.yaml and pan-cn-hpa-mp.yaml for <HPA-NAME> as provided in the pn-cn-mgmt-configmap.yaml
onutilization, dppansessionsslproxyutilization, dppanthroughput, dppanpacketrate, dppanconnectionspersecond
  - update the pan-cn-hpa-dp.yaml with all metrics to be used for scaling dp, provided yaml contains all the metrics being published for dp
  - update the pan-cn-hpa-mp.yaml with all metrics to be used for scaling mp, provided yaml contains all the metrics being pu
blished for dp
  - provided pan-cn-hpa-mp.yaml and pan-cn-hpa-dp.yaml contain scaling policies which ensure 1 pod is being scaled up/down at a time, update the policies for scaling if more pods needs to be scaled. Value provided for periodSeconds to scaleup is base
d on max time pod takes to be in ready state
GKE
```
kubectl apply -f pan-cn-adapter.yaml
kubectl apply -f pan-cn-hpa-crole.yaml
kubectl apply -f pan-cn-hpa-dp.yaml
kubectl apply -f pan-cn-hpa-mp.yaml
```

For HPA in EKS environment, it's expected users would want to customize the YAMLs as per below:
  -  Update the metrics that would be used for scaling dps in pan-cn-hpa-dp.yaml and mps pan-cn-hpa-mp.yaml.
  -  If mp is deployed in custom namespace, update the pan-cn-adapter yaml with custom namespace instead of kubesystem.
  -  pan-cn-externalmetrics.yaml contains all the metrics published for dp and mp 
  -  mp scaling metrics are mppanloggingrate, dppandataplaneslots
  -  dp scaling metrics are dpdataplanecpuutilizationpct, ,dpdataplanepacketbufferutilization, dppansessionactive, dppansessionutilization, dppansessionsslproxyutilization, dppanthroughput, dppanpacketrate, dppanconnectionspersecond
  - update the pan-cn-hpa-dp.yaml with all metrics to be used for scaling dp, provided yaml only contains single metric for example
  - update the pan-cn-hpa-mp.yaml with all metrics to be used for scaling mp, provided yaml only contains single metric for example
  - provided pan-cn-hpa-mp.yaml and pan-cn-hpa-dp.yaml contain scaling policies which ensure 1 pod is being scaled up/down at a time, update the policies for scaling if more pods needs to be scaled. Value provided for periodSeconds to scaleup is base
d on max time pod takes to be in ready state
  - allow CloudWatch complete access to both IAM roles associated with your Kubernetes pods and clusters.
  - the worker nodes’ role need to have the AWS managed policy ‘CloudwatchAgentServerPolicy’  to be able to publish the custom metrics to Cloudwatch so the HPA can pull them
EKS
```
kubectl apply -f pan-cn-adapter.yaml
kubectl apply -f pan-cn-externalmetrics.yaml
kubectl apply -f pan-cn-hpa-dp.yaml
kubectl apply -f pan-cn-hpa-mp.yaml
```

For HPA in AKS environment, it's expected users would want to customize the YAMLs as per below:
  -  Update the metrics that would be used for scaling dps in pan-cn-hpa-dp.yaml and mps pan-cn-hpa-mp.yaml.
  -  Update pan-cn-hpa-secret.yaml for Azure Application Insight specific details
  -  If mp is deployed in custom namespace, update the pan-cn-adapter yaml with custom namespace instead of kubesystem.
  -  Update the pan-cn-hpa-dp.yaml and pan-cn-hpa-mp.yaml for <HPA-NAME> as provided in the pn-cn-mgmt-configmap.yaml
AKS
  -  mp scaling metrics are mppanloggingrate, dppandataplaneslots
  -  dp scaling metrics are dpdataplanecpuutilizationpct, ,dpdataplanepacketbufferutilization, dppansessionactive, dppansessionutilization, dppansessionsslproxyutilization, dppanthroughput, dppanpacketrate, dppanconnectionspersecond
  - update the pan-cn-hpa-dp.yaml with all metrics to be used for scaling dp, provided yaml only contains single metric for example
  - update the pan-cn-hpa-mp.yaml with all metrics to be used for scaling mp, provided yaml only contains single metric for example
  - provided pan-cn-hpa-mp.yaml and pan-cn-hpa-dp.yaml contain scaling policies which ensure 1 pod is being scaled up/down at a time, update the policies for scaling if more pods needs to be scaled. Value provided for periodSeconds to scaleup is based on max time pod takes to be in ready state
```
kubectl apply -f pan-cn-hpa-secret.yaml
kubectl apply -f pan-cn-adapter.yaml
kubectl apply -f pan-cn-custommetrics.yaml
kubectl apply -f pan-cn-hpa-dp.yaml
kubectl apply -f pan-cn-hpa-mp.yaml
```

**Multus and OpenShift**

Multus CNI acts as a "meta-plugin", that calls other CNI plugins. OpenShift comes with multus enabled, so its pan-cni.yaml takes care of that.
For platforms where Multus is optional and supported e.g. self-managed (native), a separate pan-cni-multus.yaml is provided which should be used instead of pan-cni.yaml.

To make PAN-CNI plugin work with multus, these 2 extra steps are needed for the application pods:
  - A NetworkAttachmentDefinition "pan-cni" needs to be deployed in every app pod's namespace
   ```kubectl apply -f pan-cni-net-attach-def.yaml -n <target-namespace>``` 
  - An annotation ```k8s.v1.cni.cncf.io/networks: pan-cni``` in the app pod yaml

Refer to the deployment documentations for more details on it.

**Custom CA certificate support**

For using custom signing certificate create a secret to pass the CA certificates(including all the intermediary certs) to the mgmt and the ngfw pod
```kubectl -n <CN-series namespace> create secret generic custom-ca-secret --from-file=ca.crt```
  Keep the file name as "ca.crt"
The volume for custom certificates in pan-cn-mgmt.yaml and pan-cn-ngfw.yaml is optional.

**Documentation**

- [CN-Series Deployment](<https://docs.paloaltonetworks.com/cn-series/10-0/cn-series-deployment.html>)
- [CN-Series Datasheet](<https://www.paloaltonetworks.com/resources/datasheets/cn-series-container-firewall>)

