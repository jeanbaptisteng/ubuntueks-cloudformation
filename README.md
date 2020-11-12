# **Ubuntu EKS CloudFormation Script for extra package installation for Amazon EKS** #


This is an unoffical Cloudformation template to build Ubuntu-based of EKS node group with extra system package (ie: Glusterfs, nfs...) installed. 

Most of this is one-time setup. If you already have an EKS cluster, and familar with Cloudformation, please download the template and skip to the last step.

This Cloudformation template was written based-on official Amazon Linux CloudFormation Template.  

# Dependencies

The tools required to implement the whole cluster and workernode group is very simple. Just the latest version of [awscli](https://aws.amazon.com/cli/) and web browser that can access AWS WebConsole is enough. 


# Downloads 

Both CloudFormation Template for *on-demand* instances autoscaling group and *spot* instances autoscaling group are availiable here: 

**Yes, we support SPOT instances!!!**

**[On-Demand Instance](https://raw.githubusercontent.com/jeanbaptisteng/ubuntueks-cloudformation/master/Ubuntu.yaml)**
 
**[SPOT Instance](https://raw.githubusercontent.com/jeanbaptisteng/ubuntueks-cloudformation/master/Ubuntu-spot.yaml)**


# Quickstart
**New EKS cluster Installation**

1. Create EKS-role that include the follow permission to allow EKS cluster to manage resources in EKS (ie: arn:aws:iam::XXXXXXXXXX:role/eksServiceRole)

		arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
		arn:aws:iam::aws:policy/AmazonEKSServicePolicy
		arn:aws:iam::aws:policy/AmazonEKSVPCResourceController

2. Create EC2-role that include the follow permission in order to create and control the EKS cluster on the baston host (ie: arn:aws:iam::XXXXXXXXX:role/eks-controller-prd)

	Policy 1:
	{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:*",
                "eks:*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "iam:*",
            "Resource": "arn:aws:iam::XXXXXXXXXX:role/eksServiceRole"
        }
    ]
	}

	Policy 2: 
	AmazonEKSClusterPolicy



3. Execute the following command to create a new EKS private cluster

		aws eks create-cluster --name [EKS cluster name] \ 
		--role-arn "[EKS role in Step 2]" \
		--resources-vpc-config subnetIds=[Internal Subnet 1,Internal Subnet 2,Internal Subnet 3],securityGroupIds=[New SG that enable traffic within SG],endpointPublicAccess=false,endpointPrivateAccess=true --tags [Tag] --region=[AWS Region]

4. Gather the EKS Cluster information in AWS [WebConsole](https://console.aws.amazon.com/eks/home)



# Ubuntu WorkerNode Group setup

1. Open [CloudFormation Webconsole](https://console.aws.amazon.com/cloudformation/) in Web Browser

2. Click Create Stack =>  With existing resources (standard) to enter Create stack page

3. Select "Template is ready", "Upload a template file" 

4. In the dialogue, choose the template downloaded from the git

5. Check the Ubuntu EKS AMI ID from Official [link](https://cloud-images.ubuntu.com/docs/aws/eks/)
 
6. Fill in the blanks and select the desired size of the EC2 instance of the EKS WorkerNode group

7. Confirm the information before creating resources related to the CloudFormation Template. 

8. Check the new IAM role attached on new workernode in EC2 webconsole (for new workernode group only)


		# aws-auth-cm.yaml
		apiVersion: v1
		kind: ConfigMap
		metadata:
		  name: aws-auth
		  namespace: kube-system
		data:
	  	  mapRoles: |
    		- rolearn: <IAM role of the workernode>
      		  username: system:node:{{EC2PrivateDNSName}}
      		  groups:
        	   - system:bootstrappers
        	   - system:nodes

8. Apply the configuration. This command may take a few minutes to finish

		kubectl apply -f aws-auth-cm.yaml

# License

This is licensed under GNU GENERAL PUBLIC LICENSE.

# Components enhancing Kubernetes Cluster

[**NodelocalDNS**](https://github.com/jeanbaptisteng/bottlerocket-cloudformation/tree/master/plugins/NodeLocalDNS)



# See also

**[Ubuntu on Amazon Elastic Kubernetes Service (EKS)](https://cloud-images.ubuntu.com/docs/aws/eks/)**


