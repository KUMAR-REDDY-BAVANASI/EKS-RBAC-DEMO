# EKS-RBAC-DEMO



Step1: To create an IAM User
============================

Create a new user and allow the user programmatic access by clicking on the "Programmatic access"
checkbox. You do not need any particular permission for your user to access EKS. 
You can go ahead without selecting any permission.

Step2:  Configure the AWS CLI
=============================

$ aws configure --profile kumar

AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXXXXXX

AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXX

Default region name [None]: us-east-1

Default output format [None]: text

Step3:  aws sts get-caller-identity
====================================

Once configured you can test to see if the user is properly configured using the
aws sts get-caller-identity command:

$ aws sts get-caller-identity --profile kumar


Step4: Creating a Role for the user
====================================
With your IAM user properly configured, you can go ahead and create a role for the user.
This snippet of code creates a role named eks-user-role with a modest list permission to the pods 
resource in your cluster.

apiVersion: rbac.authorization.k8s.io/v1

kind: Role

metadata:

  name: kumar-role
  
rules:

- apiGroups: [""]
- 
  resources: ["pods","secrets"]
  
  verbs: ["list","get","watch"]


Save the above snippet of code in a file and then apply the Role to your Kubernetes cluster:

$ kubectl apply -f role.yaml



Step5:  With the role configured you need to create a corresponding RoleBinding
================================================================================


kind: RoleBinding

metadata:

  name: kumar-role-binding
  
subjects:

- kind: User
- 
  name: kumar
  
  apiGroup: rbac.authorization.k8s.io
  
roleRef:

  kind: Role
  
  name: kumar-role
  
  apiGroup: rbac.authorization.k8s.io


Save the above snippet of code in a file and then apply the Role Binding to your Kubernetes cluster:

$ kubectl apply -f role-binding.yaml


STEP6:  Adding the user to the aws-auth configmap
==================================================
If you want to grant additional AWS users or roles the ability to interact with your EKS cluster, 
you must add the users/roles to the aws-auth ConfigMap within Kubernetes in the kube-system namespace.

You can do this by either editing it using the kubectl edit command:


$ kubectl edit configmap aws-auth -n kube-system


Add the user under the mapUsers as an item in the aws-auth ConfigMap:

data:

  mapUsers: |
  
    - userarn: arn:aws:iam::123456789012:user/eks-user
    
      username: kumar
      
      groups:
      
      - kumar-role


If the user is properly configured you should be able to list pods in the Cluster:

$ kubectl get pods --as kumar

The --as flag impersonates the request to Kubernetes as the given user. You can use this flag to test permissions for any given user.




Thank You,
kumar


























EXTRA PERMISSIONS FOR THE USER

'''

Configuring permissions for the user
The role which you defined previously only had permission to list pods. The eks-user cannot access any other Kubernetes resources like Deployments, ConfigMaps, Events, Secrets, logs or even shell into a given pod.

In a real-world scenario, you will need to provide permissions to a user to access the required resources. The below snippet of code provides access to resources such as events, pods, deployments, configmaps and secrets.

rules:
- apiGroups: [""]

  resources: ["events"]
  
  verbs: ["get", "list", "watch"]
  
  
- apiGroups: [""]

  resources: ["pods", "pods/log", "pods/exec"]
  
  verbs: ["list", "get", "create", "update", "delete"]
  
  
- apiGroups: ["extensions", "apps"]

  resources: ["deployments"]
  
  verbs: ["list", "get", "create", "update", "delete"]
  
  
- apiGroups: [""]

  resources: ["configmaps"]
  
  verbs: ["list", "get", "create", "update", "delete"]
  
  
- apiGroups: [""]

  resources: ["secrets"]
  
  verbs: ["list", "get", "create", "update", "delete"]
  
  
Add the above permissions to the role.yaml file and apply the changes, using kubectl apply -f.

'''

















