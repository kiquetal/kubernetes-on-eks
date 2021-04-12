##### K8s - notes from oreilly courses

#### Create subnet and vpc for the cluster

	https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

#### Create cluster

	https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

	You need to create IAM role for the cluster
	
	https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html

        For 1.17 configure VPC CNI,ussing service account

	https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html
        [execute the eksctl example]

	eksctl create iamserviceaccount \
    --name aws-node \
    --namespace kube-system \
    --cluster <cluster_name> \
    --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
    --approve \
    --override-existing-serviceaccounts
     
#### Log into a public ec2

	aws configure credentials

	Install eksctl
        curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
	Install kubectl
  
       curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

       aws eks update-kubeconfig \
	  --region us-west-2 \
  	--name my-cluster

 
#### Create nodegroups

	https://github.com/awslabs/amazon-eks-ami/blob/master/amazon-eks-nodegroup.yaml

	Search: https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html


#### Create an IAM OpenID Connect (OIDC) provider

	Select the Configuration tab.

	In the Details section, copy the value for OpenID Connect provider URL.

	Open the IAM console at https://console.aws.amazon.com/iam/.

	In the navigation panel, choose Identity Providers.

	Choose Add Provider.

	For Provider Type, choose OpenID Connect.

	For Provider URL, paste the OIDC provider URL for your cluster from step two, and then choose Get thumbprint.

	For Audience, enter sts.amazonaws.com and choose Add provider.



### Apply aws-auth

	https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html

	update instance roles with the value from the cloudformation template


### INstall AWS LOAD BALANCER

	https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/
	curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.1.2/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

eksctl create iamserviceaccount \
--cluster=<cluster-name> \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--approve



  kubectl delete pods -n kube-system -l k8s-app=aws-node

	
#### Adding IAM user to eks

	kubectl edit configmap -n kube-system aws-auth        
        mapUsers: |
	  - groups:
	      - system:masters
            usename: anotheruser
            usernameurn: aws:iam 

#### For autoscaling

	nodes ability to talk with aws:scaling, add role with correct permissions.

#### Obtain nodes

	kubectl get nodes -w

#### Describe node-group

	kubectl describe nodes


#### Get Service
	
	kubectl get service service0name -o wide


#### Create deployment from image

	kubectl create deployment autoscaler-demo --image=nginx

#### Scale deployment

	kubectl scale deployment autoscaler-demo --replicas=50	




#### Create Service account

	eksctl create iamserviceaccount \
    --name <service_account_name> \
    --namespace <service_account_namespace> \
    --cluster <cluster_name> \
    --attach-policy-arn <IAM_policy_ARN> \
    --approve \
    --override-existing-serviceaccounts


### Verify service account

	eksctl get iamserviceaccount --cluster my-cluster-two --name aws-load-balancer-controller --namespace kube-system





