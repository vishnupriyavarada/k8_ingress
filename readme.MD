
# One time activity. Done by Kuberenets admin
# Ingress Controller 

REGION_CODE=us-east-1
CLUSTER_NAME=expense
ACCOUNT_ID=529088271493

### Permissions

* 1. OIDC Provider

```
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```   
* 2. IAM Policy - We are setting permission for eks cluster to access the AWS Load Balancer

```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.12.0/docs/install/iam_policy.json

```
* 3. Create IAM role ( above step we just downloaded. Here we are applying)

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```
* 4. Provide access to EKS through IAM Policy
```
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve
```


### Install Drivers 
# Install drivers through helm

* Add the EKS chart repo to Helm

```
helm repo add eks https://aws.github.io/eks-charts
```
*  Install AWS Load Balancer Controller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
``