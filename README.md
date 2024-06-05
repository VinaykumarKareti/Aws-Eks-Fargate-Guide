# Steps to Deploy the Application in AWS EKS Using Fargate

## Prerequisites Installed on the Machine:
- `kubectl`
- `aws`
- `eksctl`

## Steps:

1. **Create a Cluster in EKS:**
    ```bash
    eksctl create cluster --name <ClusterName> --region <RegionName>
    ```

2. **Configure the EKS Cluster with `kubectl` on Your Local Machine:**
    ```bash
    aws eks update-kubeconfig --name <ClusterName> --region <RegionName>
    ```

3. **Associate IAM OIDC Provider:**
    ```bash
    eksctl utils associate-iam-oidc-provider --cluster <ClusterName> --region <RegionName> --approve
    ```

4. **Create Deployment, Service, and Ingress YAML Files:**
    - You can use the following example script:
      ```bash
      kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
      ```

5. **(Optional) Create a Fargate Profile:**
    ```bash
    eksctl create fargateprofile \
      --cluster <ClusterName> \
      --region <RegionName> \
      --name <ProfileName> \
      --namespace <Namespace>
    ```

6. **Apply the Kubernetes Configuration:**
    ```bash
    kubectl apply -f <your-configuration-file>.yaml
    ```

7. **Download the IAM Policy:**
    ```bash
    curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
    ```

8. **Create an IAM Policy in AWS:**
    ```bash
    aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://iam_policy.json
    ```

9. **Create an IAM Service Account and Attach the Policy:**
    - Note: IAMServiceAccount is crucial for assigning policies to pods (such as the ingress ALB controller).
    ```bash
    eksctl create iamserviceaccount \
      --cluster=<ClusterName> \
      --namespace=kube-system \
      --name=aws-load-balancer-controller \
      --role-name AmazonEKSLoadBalancerControllerRole \
      --attach-policy-arn=arn:aws:iam::<YourAWSAccountID>:policy/AWSLoadBalancerControllerIAMPolicy \
      --region <RegionName> \
      --approve
    ```

10. **Add and Update the EKS Helm Repository:**
    ```bash
    helm repo add eks https://aws.github.io/eks-charts
    helm repo update eks
    ```

11. **Install the AWS Load Balancer Controller:**
    ```bash
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
      --set clusterName=<ClusterName> \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller \
      --set region=<RegionName> \
      --set vpcId=<YourVPCID>
    ```

12. **Verify the Deployment:**
    ```bash
    kubectl get deployment -n kube-system aws-load-balancer-controller
    ```

Make sure to replace placeholders like `<ClusterName>`, `<RegionName>`, `<ProfileName>`, `<Namespace>`, `<YourAWSAccountID>`, and `<YourVPCID>` with your actual values.
