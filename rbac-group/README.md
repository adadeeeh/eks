# Role Based Access Control (RBAC) in Amazon EKS

If you have different teams which needs different kind of cluster access, it would be difficult to manually add or remove access for each EKS cluster.

We can leverage on AWS IAM Groups and IAM Roles to easily add or remove users and give them permission to whole cluster or part of it depending on which groups they belong to.

We will create 3 IAM roles that we will map to 3 IAM groups.

# Steps

1.  Create IAM Roles

    We are going to create 3 roles:

    - **k8sAdmin** role which will have **admin** access in our EKS cluster
    - **k8sProd** role which will have access to **production** namespace in our EKS cluster
    - **k8sDev** role which will have access to **development** namespace in our EKS cluster

    Create roles:

    ```
    POLICY=$(echo -n '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":"arn:aws:iam::ACCOUNT_ID:root"},"Action":"sts:AssumeRole","Condition":{}}]}')

    echo POLICY=$POLICY

    aws iam create-role \
    --role-name k8sAdmin \
    --description "Kubernetes administrator role (for AWS IAM Authenticator for Kubernetes)." \
    --assume-role-policy-document "$POLICY" \
    --output text \
    --query 'Role.Arn'

    aws iam create-role \
    --role-name k8sProd \
    --description "Kubernetes developer role (for AWS IAM Authenticator for Kubernetes)." \
    --assume-role-policy-document "$POLICY" \
    --output text \
    --query 'Role.Arn'

    aws iam create-role \
    --role-name k8sDev \
    --description "Kubernetes developer role (for AWS IAM Authenticator for Kubernetes)." \
    --assume-role-policy-document "$POLICY" \
    --output text \
    --query 'Role.Arn'
    ```

    Because the above roles are only used to authenticate within our EKS cluster, they don’t need to have AWS permissions. We will only use them to allow some IAM groups to assume this role in order to have access to our EKS cluster.

2.  Create IAM Groups

    We want to have different IAM users which will be added to specific IAM groups in order to have different rights in the kubernetes cluster.

    We will create 3 groups:

    - **k8sAdmin** - users from this group will have admin rights on the kubernetes cluster
    - **k8sProd** - users from this group will have access to production namespace in our EKS cluster
    - **k8sDev** - users from this group will have access to development namespace in our EKS cluster
      <br>
      <br>

    1.  Create k8sAdmin IAM Group

        The **k8sAdmin** group will be allowed to assume the **k8sAdmin** IAM Role

        ```
        aws iam create-group --group-name k8sAdmin
        ```

        Add policy on our group which allow users from this group to assume k8sAdmin role

        ```
        ADMIN_GROUP_POLICY=$(echo -n '{
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "AllowAssumeOrganizationAccountRole",
                    "Effect": "Allow",
                    "Action": "sts:AssumeRole",
                    "Resource": "arn:aws:iam::ACCOUNT_ID:role/k8sAdmin"
                }
            ]
        }')

        echo ADMIN_GROUP_POLICY=$ADMIN_GROUP_POLICY

        aws iam put-group-policy \
        --group-name k8sAdmin \
        --policy-name k8sAdmin-policy \
        --policy-document "$ADMIN_GROUP_POLICY"

        ```

    2.  Create k8sProd IAM Group

        The **k8sProd** group will be allowed to assume the **k8sProd** IAM Role

        ```
        aws iam create-group --group-name k8sProd
        ```

        Add policy on our group which allow users from this group to assume k8sProd role

        ```
        PROD_GROUP_POLICY=$(echo -n '{
        "Version": "2012-10-17",
        "Statement": [
            {
            "Sid": "AllowAssumeOrganizationAccountRole",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::ACCOUNT_ID:role/k8sProd"
            }
        ]
        }')

        echo PROD_GROUP_POLICY=$PROD_GROUP_POLICY

        aws iam put-group-policy \
        --group-name k8sProd \
        --policy-name k8sProd-policy \
        --policy-document "$PROD_GROUP_POLICY"
        ```

    3.  Create k8sDev IAM Group

        The **k8sDev** group will be allowed to assume the **k8sDev** IAM Role

        ```
        aws iam create-group --group-name k8sDev
        ```

        Add policy on our group which allow users from this group to assume k8sDev role

        ```
        DEV_GROUP_POLICY=$(echo -n '{
        "Version": "2012-10-17",
        "Statement": [
            {
            "Sid": "AllowAssumeOrganizationAccountRole",
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::ACCOUNT_ID:role/k8sDev"
            }
        ]
        }')

        echo DEV_GROUP_POLICY=$DEV_GROUP_POLICY

        aws iam put-group-policy \
        --group-name k8sDev \
        --policy-name k8sDev-policy \
        --policy-document "$DEV_GROUP_POLICY"
        ```

3.  Create IAM Users

    In order to test our scenarios, we will create 3 users, one for each group we created

    ```
    aws iam create-user --user-name PaulAdmin
    aws iam create-user --user-name JeanProd
    aws iam create-user --user-name PierreDev
    ```

    Add users to associated groups

    ```
    aws iam add-user-to-group --group-name k8sAdmin --user-name PaulAdmin
    aws iam add-user-to-group --group-name k8sProd --user-name JeanProd
    aws iam add-user-to-group --group-name k8sDev --user-name PierreDev
    ```

    <!-- Retrieve Access Keys for fake users -->

    <!-- ```
    aws iam create-access-key --user-name PaulAdmin | tee PaulAdmin.json
    aws iam create-access-key --user-name JeanProd | tee JeanProd.json
    aws iam create-access-key --user-name PierreDev | tee PierreDev.json
    ``` -->

    Recap:

    - **PaulAdmin** is in the **k8sAdmin** group and will be able to assume the **k8sAdmin** role
    - **JeanProd** is in the **k8sProd** grpup and will be able to assume the **k8sProd** role
    - **PierreDev** is in the **k8sDev** group and will be able to assume the **k8sDev** role

4.  Configure Kubernetes RBAC

    Create kubernetes namespace

    ```
    kubectl create namespace production
    kubectl create namespace development
    ```

    Create Role and RoleBinding for production namespace and development namespace

    ```
    kubectl apply -f prod-role.yaml
    kubectl apply -f dev-role.yaml
    ```

5.  Configure Kubernetes Role Access

    In order to give access to the IAM Roles we defined previously to our EKS cluster, we need to add specific **mapRoles** to the **aws-auth** ConfigMap \
    The Advantage of using Role to access the cluster instead of specifying directly IAM users is that it will be easier to manage: we won’t have to update the ConfigMap each time we want to add or remove users, we will just need to add or remove users from the IAM Group and we just configure the ConfigMap to allow the IAM Role associated to the IAM Group.

    Update the aws-auth ConfigMap to allow our IAM Roles

    The **aws-auth** ConfigMap from the kube-system namespace must be edited in order to allow or delete arn Groups.

    This file makes the mapping between IAM Role and k8S RBAC rights. We can edit it manually or edit it using eksctl.

    ```
    eksctl create iamidentitymapping \
    --cluster cluster-1 \
    --arn arn:aws:iam::ACCOUNT_ID:role/k8sProd \
    --username prod-user

    eksctl create iamidentitymapping \
    --cluster cluster-1 \
    --arn arn:aws:iam::ACCOUNT_ID:role/k8sDev \
    --username dev-user

    eksctl create iamidentitymapping \
    --cluster cluster-1 \
    --arn arn:aws:iam::ACCOUNT_ID:role/k8sAdmin \
    --username admin \
    --group system:masters
    ```

    It can also be used to delete entries

    ```
    eksctl delete iamidentitymapping --cluster cluster-1 --arn arn:aws:iam::ACCOUNT_ID:role/k8sProd
    ```

    Run command vellow to print config map

    ```
    kubectl get cm -n kube-system aws-auth -o yaml
    ```

    You should have the config map looking something like

    ```
    apiVersion: v1
    data:
    mapRoles: |
        - groups:
        - system:bootstrappers
        - system:nodes
        rolearn: arn:aws:iam::ACCOUNT_ID:role/eksctl-cluster-1-nodegroup-ng-1-NodeInstanceRole-1VO2PR88PUHMI
        username: system:node:{{EC2PrivateDNSName}}
        - rolearn: arn:aws:iam::ACCOUNT_ID:role/k8sProd
        username: prod-user
        - rolearn: arn:aws:iam::ACCOUNT_ID:role/k8sDev
        username: dev-user
        - groups:
        - system:masters
        rolearn: arn:aws:iam::ACCOUNT_ID:role/k8sAdmin
        username: admin
    mapUsers: |
        []
    kind: ConfigMap
    ```

    We can use eksctl to get a list of all identities managed in our cluster

    ```
    eksctl get iamidentitymapping --cluster cluster-1
    ```

    Here we have created:

    - a RBAC role for K8sAdmin, that we map to admin user and give access to **system:masters** kubernetes Groups (so that it has Full Admin rights)
    - a RBAC role for k8sProd that we map on prod-user in production namespace
    - a RBAC role for k8sdev that we map on dev-user in development namespace

6.  Test EKS Access

    Create new **KUBECONFIG** file for k8sProd, k8sDev and k8sAdmin role

    ```
    eksctl utils write-kubeconfig --cluster=cluster-1 --kubeconfig=kubeconfig-prod
    eksctl utils write-kubeconfig --cluster=cluster-1 --kubeconfig=kubeconfig-dev
    eksctl utils write-kubeconfig --cluster=cluster-1 --kubeconfig=kubeconfig-admin
    ```

    - Test k8sProd role\
      Add argument bellow in args field for kubeconfig-prod

      ```
      - --role
      - arn:aws:iam::ACCOUNT_ID:role/k8sProd
      ```

      Export the KUBECONFIG file

      ```
      export KUBECONFIG=kubeconfig-prod
      ```

      Create deployment

      ```
      kubectl create deployment nginx --image=nginx -n production
      ```

      Get deployment and pod

      ```
      kubectl get pod,deploy,svc -n production
      ```

      Try get deployment and pod in another namespace

      ```
      kubectl get pod,deploy,svc -n development
      ```

      ```
      Error from server (Forbidden): pods is forbidden: User "prod-user" cannot list resource "pods" in API group "" in the namespace "development"
      Error from server (Forbidden): deployments.apps is forbidden: User "prod-user" cannot list resource "deployments" in API group "apps" in the namespace "development"
      ```

    - Test k8sDev role\
      Add argument bellow in args field for kubeconfig-dev

      ```
      - --role
      - arn:aws:iam::ACCOUNT_ID:role/k8sDev
      ```

      Export the KUBECONFIG file

      ```
      export KUBECONFIG=kubeconfig-dev
      ```

      Create deployment

      ```
      kubectl create deployment nginx --image=nginx -n development
      ```

      Get deployment and pod

      ```
      kubectl get pod,deploy -n development
      ```

      Try get deployment and pod in another namespace

      ```
      kubectl get pod,deploy -n production
      ```

      ```
      Error from server (Forbidden): pods is forbidden: User "dev-user" cannot list resource "pods" in API group "" in the namespace "production"
      Error from server (Forbidden): deployments.apps is forbidden: User "dev-user" cannot list resource "deployments" in API group "apps" in the namespace "production"
      ```

    - Test k8sAdmin role\
       Add argument bellow in args field for kubeconfig-admin

      ```
      - --role
      - arn:aws:iam::ACCOUNT_ID:role/k8sAdmin
      ```

      Export the KUBECONFIG file

      ```
      export KUBECONFIG=kubeconfig-admin
      ```

      Create deployment

      ```
      kubectl create deployment nginx --image=nginx
      ```

      Get deployment and pod in all namespace

      ```
      kubectl get pod,deploy -A
      ```

      ```
      NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
      default       pod/nginx-6799fc88d8-ht9jj     1/1     Running   0          8s
      development   pod/nginx-6799fc88d8-s7fpq     1/1     Running   0          6m23s
      kube-system   pod/aws-node-29pgz             1/1     Running   0          6h54m
      kube-system   pod/aws-node-cn4tj             1/1     Running   0          6h55m
      kube-system   pod/aws-node-dmj8z             1/1     Running   0          6h55m
      kube-system   pod/coredns-55b9c7d86b-dv9kp   1/1     Running   0          7h6m
      kube-system   pod/coredns-55b9c7d86b-w7c4r   1/1     Running   0          7h6m
      kube-system   pod/kube-proxy-dh5sm           1/1     Running   0          6h55m
      kube-system   pod/kube-proxy-gqk7q           1/1     Running   0          6h54m
      kube-system   pod/kube-proxy-rr9wf           1/1     Running   0          6h55m
      production    pod/nginx-6799fc88d8-8ngj8     1/1     Running   0          12m

      NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
      default       deployment.apps/nginx     1/1     1            1           8s
      development   deployment.apps/nginx     1/1     1            1           6m23s
      kube-system   deployment.apps/coredns   2/2     2            2           7h6m
      production    deployment.apps/nginx     1/1     1            1           12m
      ```

7.  Cleanup

    ```
    unset KUBECONFIG

    kubectl delete namespace production development
    kubectl delete deployment nginx

    eksctl delete iamidentitymapping --cluster cluster-1 --arn arn:aws:iam::ACCOUNT_ID:role/k8sAdmin
    eksctl delete iamidentitymapping --cluster cluster-1 --arn arn:aws:iam::ACCOUNT_ID:role/k8sProd
    eksctl delete iamidentitymapping --cluster cluster-1 --arn arn:aws:iam::ACCOUNT_ID:role/k8sDev

    aws iam remove-user-from-group --group-name k8sAdmin --user-name PaulAdmin
    aws iam remove-user-from-group --group-name k8sProd --user-name JeanProd
    aws iam remove-user-from-group --group-name k8sDev --user-name PierreDev

    aws iam delete-group-policy --group-name k8sAdmin --policy-name k8sAdmin-policy
    aws iam delete-group-policy --group-name k8sProd --policy-name k8sProd-policy
    aws iam delete-group-policy --group-name k8sDev --policy-name k8sDev-policy

    aws iam delete-group --group-name k8sAdmin
    aws iam delete-group --group-name k8sProd
    aws iam delete-group --group-name k8sDev

    aws iam delete-user --user-name PaulAdmin
    aws iam delete-user --user-name JeanProd
    aws iam delete-user --user-name PierreDev

    aws iam delete-role --role-name k8sAdmin
    aws iam delete-role --role-name k8sProd
    aws iam delete-role --role-name k8sDev
    ```

# References:

1. https://www.eksworkshop.com/beginner/091_iam-groups/
2. https://medium.com/swlh/kubernetes-access-control-with-iam-and-rbac-e9c051ee226b
3. https://jicowan.medium.com/rbac-walkthrough-using-a-iam-group-to-assign-permission-to-the-kubernetes-api-ebd4014ed91d
4. https://aws-blog.de/2021/08/iam-what-happens-when-you-assume-a-role.html
