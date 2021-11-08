# CN-Series Next-Generation Firewall Deployment

This is a repository for YAMLs to deploy CN-Series Next-Generation firewall from Palo Alto Networks.

All the YAMLs required to deploy CN-Series on a given cloud platform are present under that cloud platform specific directory. Users can use these YAMLs as is to deploy CN-Series quickly after filling in just these fields from their setup:
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
      # Intended License Bundle type - "CN-X-BASIC", "CN-X-BND1", "CN-X-BND2"
      # based on the authcode applied on the Panorama K8S plugin
      PAN_BUNDLE_TYPE: <license-bundle-type>
  ```
For production deployment, it's expected users would want to customize the YAMLs as per below:
  - Resources (cpu, memory) fields in pan-cn-mgmt.yaml and pan-cn-ngfw.yaml are pre-populated but should be customized to better suit the deployment scenario.
  - There are some optional fields in the configmaps which users can add e.g. PAN_PANORAMA_IP2 for Panorama in HA, or CLUSTER_NAME for easier identification when managing multiple Kubernetes clusters under the same Panorama.
  **Note**: For complex setup and advanced topics needing modifications in the YAMLs, refer to the deployment documentations for details. Changing a field might require modification in multiple places and multiple YAMLs.


Once the YAMLs have been modified as desired, these YAMLs can be deployed as:
``` 
kubectl apply -f plugin-serviceaccount.yaml
kubectl apply -f pan-cni-serviceaccount.yaml
kubectl apply -f pan-mgmt-serviceaccount.yaml
kubectl apply -f pan-cni-configmap.yaml
kubectl apply -f pan-cni.yaml
kubectl apply -f pan-cn-mgmt-secret.yaml
kubectl apply -f pan-cn-mgmt-configmap.yaml
kubectl apply -f pan-cn-mgmt.yaml
kubectl apply -f pan-cn-ngfw-configmap.yaml
kubectl apply -f pan-cn-ngfw.yaml
```

To enable the security for the application pods, apply the following annotation to their YAMLs, OR, to enable the security for all the pods in a given namespace, apply this annotation to the namespace: ```     paloaltonetworks.com/firewall: pan-fw``` e.g. for "default" namespace 
```
kubectl annotate namespace default paloaltonetworks.com/firewall=pan-fw
```

**OpenShift**

OpenShift has multus CNI acting as a "meta-plugin", that calls other CNI plugins. To make PAN-CNI plugin work with multus, these 2 extra steps are needed for the application pods:
  - A NetworkAttachmentDefinition "pan-cni" needs to be deployed in every app pod's namespace
   ```kubectl apply -f pan-cni-net-attach-def.yaml -n <target-namespace>``` 
  - An annotation ```k8s.v1.cni.cncf.io/networks: pan-cni``` in app pod yaml

Refer to the deployment documentations for more details on it.

**Documentation**

- [CN-Series Deployment](<https://docs.paloaltonetworks.com/cn-series/10-0/cn-series-deployment.html>)
- [CN-Series Datasheet](<https://www.paloaltonetworks.com/resources/datasheets/cn-series-container-firewall>)
