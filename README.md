Steps to deploy the Application in Aws EKS using Fargate
----------------------------------------------------------
----------------------------------------------------------

Prerequisites that are installed in machine :
---------------------------------------------
 - kubectl
 - aws
 - eksctl

Steps:
------

 - First to create a cluster in EKS
   eksctl create cluster --name <ClusterName> --region <RegionName>
 - to config the eks cluster kubectl in our local machine.
   aws eks update-kubeconfig --name <ClusterName> --region <RegionName>
 - eksctl utils associate-iam-oidc-provider --cluster <ClusterName> --region <RegionName> --approve
 - Create the deployment.yaml , service.yaml and ingress.yaml (Example script should be given in this repo).
   example script be
   kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
 - If you want to create any fargate profile.(Optional Step)
   eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048  
 - Use the kubectl apply command.
 - Now download the IAM policy.
   curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
 - Creating a Iam Policy in Aws using the below command.
   aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
 - Now creating a IAMServiceAccound and attaching the policy to it.(IamServiceAccount is very important to create while assing any policies to the pods(I mean ingress alb controller))
   eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
   --region <RegionName>
  --approve
 - helm repo add eks https://aws.github.io/eks-charts
 - helm repo update eks
 - helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
 - kubectl get deployment -n kube-system aws-load-balancer-controller
   
