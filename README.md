Step 1 - [ FRONTEND ]
- Write Docker File For Backend and frontend environment
- sudo apt install docker.io
- chown $USER /var/run/docker.sock
- docker build Docker-frontend . ( Docker Front end image should be created )
- docker run -d -p 
- docker run -d -p 3000:3000 frontend-docker:latest ( give th port number to docker image with repository name with tag )
- Allow 3000 port in aws EC2 inbound rule 

- Now push that image to AWS ECR ( for that you need to have aws cli installed on your pc to install commands are :)
-curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
-sudo apt install unzip
-unzip awscliv2.zip
-sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
-aws configure

- after installing aws cli u need aws iam user that will have all administrator permissions(go to iam in aws and add user and give all permisions )
- now create a security crediantials for that user and create access key 
- aws configure write this and give id and key 
- now create a ECR repository and go to it and click on view push commands and apply all push commands 
- done image pushed to ecr 

Step 2 - [ Backend ]
- docker build docker-backend ( It should build docker backend image )
- docker run -d -p 3500:3500 docker-backend:latest ( give th port number to docker image with repository name with tag )
- run all ecr push command to push on aws ecr 

Step 3 - [ Installation of eks and kubectl ]

Install kubectl
-curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
-chmod +x ./kubectl
-sudo mv ./kubectl /usr/local/bin
-kubectl version --short --client

Install eksctl
-curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
-sudo mv /tmp/eksctl /usr/local/bin
-eksctl version

 Step 4 -
 Setup EKS Cluster
-eksctl create cluster --name three-tier-cluster --region us-east-1 --node-type t2.micro --nodes-min 2 --nodes-max 2
-aws eks update-kubeconfig --region us-east-1 --name three-tier-cluster (Get only specified region cluster)
-kubectl get nodes ( to get created clusters )

Step 5 -
-We need to create kubernetics Manifest 
-If you want kubernrics deployment search on google everythiong already pre written just copy paste and add as per your project and deploy 
-just change image in kubermetics manifest 
-we need workspace for that
-kubectl create namespace three-tier
-in database file and namespace is interlinked 
-we will deploy mongodb now 
-kubectl apply -f deployment.yaml
-kubectl apply -f secrets.yaml
-kubectl apply -f service.yaml
-kubectl apply -f pvc.yaml
-kubectl apply -f pv.yaml
-kubectl get svc -n three-tier (To get servive is stared or not Types of service - nodeport , load balancer , cluster ip)
-kubectl get pods -n three-tier
-kubectl logs api-6f666fd947-4g6zs -n three-tier ( It must show connected to database)
-apply same for frontend
-we are building with kubernetics beacuse autoscaling , auto healing ..etc 

Step 6 -
-Through Helm We will install ingress controller ( it will manage internal networking)
-curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
-aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
-eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=three-tier-cluster --approve

Step 7 -
-Installation of ALB 
-sudo snap install helm --classic
-helm repo add eks https://aws.github.io/eks-charts
-helm repo update eks
-helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
-kubectl get deployment -n kube-system aws-load-balancer-controller
-kubectl apply -f full_stack_lb.yaml
-kubectl apply -f ingress.yaml
-kubectl get ing -n three-tier

Step 8 -
-Add Address of eks to domain and access the site done .
-Please do with T2.medium instance minimum














