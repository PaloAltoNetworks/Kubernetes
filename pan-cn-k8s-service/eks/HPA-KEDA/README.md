### CN-series v2 HPA using KEDA

Current implementation of HPA on AWS and AZR is reliant on custom metrics adapter provided by the respective CSP. But with kubernetes 1.22, kubernetes autoscaling/v2beta1 is deprecated. What this means to us is that kubernetes APIs the custom metrics adapter uses will no longer be supported. 

In order to continue supporting custom metrics based HPA, CSP recommends KEDA as the way forward.

##Enable HPA on cn-series
- Ensure that the HPA params have been filled out in the pan-cn-mgmt-configmap.yam to enable HPA
- Ensure PAN_NAMESPACE_EKS field has a unique name across your AWS account in your region. This avoids overwriting metrics from different CN clusters with the same EKS namespace.

##AWS IAM role setup for Cloudwatch access

#Access for MP to publish metrics to Cloudwatch
Add "CloudWatchFullAccess" policy to the node IAM role.

#Access for KEDA to retrieve metrics frpm Cloudwatch
 Associate an IAM role with the keda operator service account. This is done by annotating the role-arn in the keda service account. This will enable only the keda service account to gain access to the Cloudwatch not the entire node on which keda is running.
-[Create an IAM OIDC provider for your cluster](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) – You only need to do this once for a cluster.
-[Create an IAM role and attach an IAM policy](https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html) to it with the permissions that your service accounts need.Provide Cloudwatch access policy
-[Associate an IAM role with a service account](https://docs.aws.amazon.com/eks/latest/userguide/specify-service-account-role.html) – Complete this task for each Kubernetes service account that needs access to AWS resources
-Modify the keda-2.7.1.yaml and look for keda-operator serviceaccount and update the role arn in the annotation.

##Deploy the KEDA pods
- Deploy the keda pods using 'kubectl apply -f keda-2.7.1.yaml'
- mp scaling metrics are mppanloggingrate, dppandataplaneslots
- dp scaling metrics are dpdataplanecpuutilizationpct, ,dpdataplanepacketbufferutilization, dppansessionactive, dppansessionutilization, dppansessionsslproxyutilization, dppanthroughput, dppanpacketrate, dppanconnectionspersecond
- Modify the pan-cn-hpa-mp-scaled-obj.yaml with the metrics to be used for scaling MP
- Modify the pan-cn-hpa-mp-scaled-obj.yaml with the metrics to be used for scaling MP
- In the scaled-obj yaml, update the following fields 
--minReplicaCount
--maxReplicaCount
--namespace field should match the PAN_NAMESPACE_EKS in pan-cn-mgmt-configmap.yaml with '_dimensions' suffix
--awsRegion
--metricName
