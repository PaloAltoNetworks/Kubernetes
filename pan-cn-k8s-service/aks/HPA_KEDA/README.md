#### HORIZONTAL POD AUTOSCALING ON CNV2 USING KEDA

Starting Kubernetes 1.22, kubernetes autoscaling/v2beta1 is deprecated. To continue supporting Horizontal Pod Autoscaling based on custom metrics, CSP recommends using KEDA.

### Enable HPA on cn-series
- Deploy an Azure Application Insights instance in your cluster and provide the instrumentation key and Azure Application Insight ID as a K8s secret.

- Create a service principal with the App ID, go to subscriptions and assign it a role.

- Make sure that the following HPA params have been filled out in pan-cn-mgmt-configmap.yaml file
  #PAN_CLOUD: "AKS"
  #HPA_NAME: "<name>" #unique name to identify hpa resource per namespace or per tenant 
  #PAN_INSTRUMENTATION_KEY: "<>" #Azure APP Insight Instrumentation Key
  #PUSH_INTERVAL: "15" #time interval to publish metrics to azure app insight

- Add the following params in scaled_obj.yaml file
  appinsights-appid: "<Azure App Insight Application ID obtained from API Access>"
  azure-client-id: "<Azure SP APP ID associated with corresponding resource group with monitoring reader access>"
  azure-client-secret: "<Azure SP Password associated with corresponding resource group with monitoring reader access>"
  azure-tenant-id: "<Azure SP tenant ID associated with corresponding resource group with monitoring reader access>"


### Deploy the KEDA pods
- Deploy the KEDA pods using "kubectl apply -f keda-2.9.0.yaml".
- Deploy the scaled_obj.yaml file for scaling MP pods "kubectl apply -f pan-cn-hpa-mp-scaled-obj.yaml".
- Deploy the scaled_obj.yaml file for scaling DP pods "kubectl apply -f pan-cn-hpa-dp-scaled-obj.yaml".
- MP scaling metrics are panloggingrate, pandataplaneslots.
- DP scaling metrics are dataplanecpuutilizationpct, dataplanepacketbufferutilization, pansessionactive, pansessionutilization,
  pansessionsslproxyutilization, panthroughput, panpacketrate, panconnectionspersecond.
- Modify the scaled_obj.yaml files to add minReplicaCount and maxReplicaCount.
- Modify the pan-cn-hpa-mp-scaled-obj.yaml file with the metrics to be used for scaling MP
- Modify the pan-cn-hpa-dp-scaled-obj.yaml file with the metrics to be used for scaling DP.
 
