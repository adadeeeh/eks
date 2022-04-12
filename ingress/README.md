# EKS Ingress

This EKS use ALB ingress. **Please read the references for more knowledge**

# Steps

1. Create EKS cluster with IAM OIDC for the cluster

   ```
   eksctl create cluster -f eks-cluster.yaml
   ```

2. Install AWS Load Balancer controller

   ```
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=app-lb-demo --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
   ```

3. Create 2 deployment for the sake of testing

   ```
   kubectl apply -f apple-deployment.yaml
   kubectl apply -f banana-deployment.yaml
   ```

4. Create ingress

   ```
   kubectl apply -f fruit-ingress.yaml
   ```

5. Test

   - Access **ADDRESS**/apple
   - Access **ADDRESS**/banana

6. Cleanup

   - Delete ingress

     ```
     kubectl delete -f fruit-ingress.yaml
     ```

   - Delete deployment

     ```
     kubectl apply -f apple-deployment.yaml
     kubectl apply -f banana-deployment.yaml
     ```

   - Delete cluster

     ```
     eksctl delete cluster -f eks-cluster.yaml
     ```

# References

1. https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
2. https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html#application-load-balancer-sample-application
3. https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/deploy/installation/
4. https://www.linkedin.com/pulse/deploying-aws-load-balancer-controller-ingress-eks-kai-jian-tan/
5. https://blog.sivamuthukumar.com/aws-load-balancer-controller-on-eks-cluster
6. https://github.com/k21academyuk/EKS-Ingress-Controller
