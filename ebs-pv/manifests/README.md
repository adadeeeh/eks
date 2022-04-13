# EBS Dynamic Persistent Volume

EBS persistent volume dynamically provision. Using Amazon CSI Driver

# Prerequisites

1. IAM OIDC provider for the cluster
2. Amazon EBS CSI driver IAM role

# Step

1. Create cluster with IAM OIDC and EBS CSI IAM Role Service Account

   ```
   eksctl create cluster -f eks-cluster.yaml
   ```

2. Add the Amazon EBS CSI add-on

   ```
   eksctl create addon --name aws-ebs-csi-driver --cluster cluster-1 --service-account-role-arn arn:aws:iam::ACCOUNT_ID:role/AmazonEKS_EBS_CSI_DriverRole --force
   ```

   Or

   ```
   kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.5"
   ```

   Verify driver is running:

   ```
   kubectl get pods -n kube-system
   ```

3. Testing

   - Create pod with persistent volume

     ```
     kubectl apply -f manifests
     ```

   - List peristent volumes in default namespace

     ```
     kubectl get pv
     ```

     Example output:

     ```
     NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
     pvc-a2095714-35f5-4ca6-b0d0-076d7933b386   4Gi        RWO            Delete           Bound    default/ebs-claim   ebs-sc                  33s
     ```

     `WaitForFirstConsumer` means that it will not provision persistent volume until pod using the persistent volume claim is created

4. Cleanup

   - Delete pod and persisten volume

     ```
     kubectl delete -f manifests
     ```

   - Remove Amazon EBS CSI add-on

     ```
     kubectl delete -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.5"
     ```

   - Delete cluster

     ```
     eksctl delete cluster -f eks-cluster.yaml
     ```

# References

1. https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html
2. https://github.com/kubernetes-sigs/aws-ebs-csi-driver
3. https://www.kubermatic.com/blog/keeping-the-state-of-apps-5-introduction-to-storage-classes/
