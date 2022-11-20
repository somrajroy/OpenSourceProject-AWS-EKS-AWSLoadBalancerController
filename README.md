### AWS-EKS-Ingress-AWSLoadbalancerController <br/>
AWS EKS Ingress with latest AWS Load Balancer Controller<br/>
* Clone the repository and navigate to the folder lab-05
* It is highly recommended to use eksctl for cluster creation and subsequent management as eksctl will automatically do many tasks which has to be manually done otherwise <br/>
* Create an EKS Cluster <br/>
  $ eksctl create cluster --name k8sdemo --version 1.23 --region us-west-2 --nodegroup-name k8snodes --node-type t3.medium --nodes 2 <br/>
* An existing AWS Identity and Access Management (IAM) OpenID Connect (OIDC) provider for your cluster. To determine whether one already exists, or to create one, refer [Creating an IAM OIDC provider for your cluster](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html).<br/>
* Create the IAM policy "AWSLoadBalancerControllerIAMPolicy" <br/>
  $ aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
* Create service account and role. [Refer relvant sections of the document and make necessary changes](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) <br/>
  $ eksctl create iamserviceaccount --cluster=k8sdemo --namespace=kube-system --name=aws-load-balancer-controller --role-name "AmazonEKSLoadBalancerControllerRole" --attach-policy-arn=arn:aws:iam::127538279091:policy/AWSLoadBalancerControllerIAMPolicy --approve <br/>
* Install the controller using Helm <br/>
  $ helm repo add eks https://aws.github.io/eks-charts <br/>
* Update the repo <br/>
  $ helm repo update <br/>
* [Refer every section of this document and make necessary changes. If you are using eksctl then below changes will suffice for demo purposes](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html) <br/>
  * Go to EC2 console and check the IAM role assigned to the EC2 instances. Add an inline policy by coping the JSON code from "ECRALB.json".<br/>
  * Add the AWS managed policy "AmazonEC2FullAccess" to the role <br/>
* Install and upgrade the controller <br/>
  $ helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=k8sdemo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller <br/>
  $ helm upgrade aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=k8sdemo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller <br/>
* Verify that the controller is installed <br/>
  $ kubectl get deployment -n kube-system aws-load-balancer-controller <br/>
* To debug view the AWS Load Balancer Controller logs. These logs might contain error messages that you can use to diagnose issues with your deployment. <br/>
  $ kubectl logs -n kube-system deployment.apps/aws-load-balancer-controller <br/> 
* Deploy the applications (pods and ClusterIP services) <br/>
  $ kubectl apply -f cats.yaml <br/>
  $ kubectl apply -f dogs.yaml <br/>
  $ kubectl apply -f birds.yaml <br/>
* Create the ingress resource <br/>
  $ kubectl apply -f ingress.yaml <br/>
* Get the DNS name of the ALB from EC2 management console or by running below command. <br/>
  $ kubectl get ingress <br/>
    ![image](https://user-images.githubusercontent.com/92582005/202916008-c84483bc-73a4-48e1-8e5c-17f5535e2208.png) <br/>
* Access the application - Browse to the cats, dogs and birds service <br/>
  $ http://<<-DNS name from above->>/cats <br/>
  $ http://<<-DNS name from above->>/dogs <br/>
  $ http://<<-DNS name from above->>/birds <br/>
* Clean up AWS enviornment <br/>
  $ eksctl delete cluster --name k8sdemo <br/>



### References <br/>
* [Application load balancing on Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)<br/>
* [Installing the AWS Load Balancer Controller add-on](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)<br/>
* [Kubernetes Ingress with AWS ALB Ingress Controller](https://aws.amazon.com/blogs/opensource/kubernetes-ingress-aws-alb-ingress-controller/)<br/>
* [How do I provide external access to multiple Kubernetes services in my Amazon EKS cluster?](https://aws.amazon.com/premiumsupport/knowledge-center/eks-access-kubernetes-services/)<br/>
* [How do I set up an Application Load Balancer using the AWS Load Balancer Controller on an Amazon EC2 node group in Amazon EKS?](https://aws.amazon.com/premiumsupport/knowledge-center/eks-alb-ingress-controller-setup/)<br/>
* [Amazon Elastic Container Registry Identity-based policy examples](https://docs.aws.amazon.com/AmazonECR/latest/userguide/security_iam_id-based-policy-examples.html)<br/>
