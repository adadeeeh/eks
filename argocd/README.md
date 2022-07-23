# Argo CD

## Description

This folder is used to try Continous Delivery using Argo CD

## Steps

1. Create K8s cluster
   ```
   eksctl create cluster -f eks-cluster.yaml
   ```
2. Push K8s manifests to GitHub

3. Install Argo CD

   ```
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

4. Download Argo CD CLI

   ```
   brew install argocd
   ```

5. Access Argo CD API Server using Load Balancer or Port Forwarding

   Service type Load Balancer

   ```
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
   ```

   Port Forwarding

   ```
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

6. Retrieve Password
   ```
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```
7. Push Argo CD CRD to GitHub

8. Apply Argo CD CRD

   ```
   kubectl apply -f application.yaml
   ```

9. Clean Up
   ```
   kubectl delete -f application.yaml
   kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   eksctl delete cluster -f eks-cluster.yaml
   ```

## References

1. https://youtu.be/MeU5_k9ssrs
2. https://argo-cd.readthedocs.io/en/stable/getting_started/
3. https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/
