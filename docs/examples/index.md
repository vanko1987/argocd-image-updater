# Examples

This chapter contains community contributed examples on how to configure Argo
CD Image Updater for a certain registry or environment.

Feel free to submit your own examples or guides to the documentation, as we do
not have access to all the registries and cloud platforms out there.

There are three requirements to setup the integration with AWS ECR:
1) Service account setup:

serviceAccount:
  # -- Specifies whether a service account should be created
  create: true
  # -- Annotations to add to the service account
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::[AWS ACCOUNT ID]:role/argocd
  # -- The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: "argocd-image-updater"
  
2) AWS Policy associated with the role to allow ECR access - e.g. :
![image](https://user-images.githubusercontent.com/80322141/158956735-372c6440-a9fc-4d44-bc34-8c55e7f9bd00.png)


3) Policy to assume web identity of EKS cluster oidc provider (for the argocd service account user) which must be added under "Trust relationship" of the role:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Ec2StsAssumeRole",
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Sid": "AllowArgoCdToAssumeSpokeRoles",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<Account-ID>:oidc-provider/<Cluster OIDC Identity Provider>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "<Cluster OIDC Identity Provider>:sub": "system:serviceaccount:argocd:argocd-image-updater"
        }
      }
    }
  ]
}
  
where Account ID and OIDC Identity Provider can be retrieved via AWS CLI as specified in this article: 
https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html
  
e.g 
  ![Untitled](https://user-images.githubusercontent.com/80322141/158957475-747cfdb7-a75d-4462-b6bc-fa5f7b610c96.png)
