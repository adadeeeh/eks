# Role Based Access Control (RBAC) in Amazon EKS

# Steps

1. Using EKS Admin user, create test namespace and deployment

   ```
   kubectl create namespace rbac-namespace
   kubectl create deployment nginx --image=nginx -n rbac-namespace
   ```

2. Verify test pod were properly installed

   ```
   kubectl get all -n rbac-namespace
   ```

3. Create IAM user

   ```
   aws iam create-user --user-name rbac-user
   aws iam create-access-key --user-name rbac-user | tee rbac-user-AccessKey.json
   ```

4. Add AWS profile

   ```
   aws configure --profile rbac-user
   ```

5. Add new user mapping for **rbac-user** user

   ```
   kubectl edit configmap aws-auth -n kube-system
   ```

   ```
   apiVersion: v1
   data:
   mapRoles: |
     - groups:
       - system:bootstrappers
       - system:nodes
       rolearn: arn:aws:iam::660346824954:role/eksctl-cluster-1-nodegroup-ng-1-NodeInstanceRole-OXQY8N55ADXG
       username: system:node:{{EC2PrivateDNSName}}
   mapUsers: |
     - userarn: arn:aws:iam::660346824954:user/rbac-user
       username: rbac-user
   kind: ConfigMap
   metadata:
   creationTimestamp: "2022-04-08T10:46:49Z"
   name: aws-auth
   namespace: kube-system
   ```

6. Create Role rbacuser-role.yaml
7. Create RoleBinding rbacuser-role-binding.yaml
8. Apply Role and RoleBinding

   ```
   kubectl apply -f rbacuser-role.yaml
   kubectl apply -f rbacuser-role-binding.yaml
   ```

9. Verify Role and RoleBinding. First, change to IAM user rbac-user

   ```
   export AWS_PROFILE=rbac-user
   ```

   Get pod in namespace rbac-namespace

   ```
   kubectl get pod -n rbac-namespace
   ```

10. Clean Up (Using EKS Admin user)

    ```
    kubectl delete namespace rbac-namespace
    aws iam delete-access-key --user-name=rbac-user --access-key-id=AccessKeyId
    aws iam delete-user --user-name rbac-user
    rm rbac-user-AccessKey.json
    ```

    Delete user mapping in step 5.

# References

1. https://www.eksworkshop.com/beginner/090_rbac/
2. https://towardsaws.com/controlling-aws-eks-access-using-aws-iam-and-kubernetes-rbac-9309d5ac4d11
3. https://docs.bitnami.com/tutorials/configure-rbac-in-your-kubernetes-cluster/
