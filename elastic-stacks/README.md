# Elastic Stacks

## Description

This folder is used to try Elastic Stacks using Elastic Cloud on Kubernetes (ECK)

## Steps

1. Create K8s cluster

   ```
   eksctl create cluster -f eks-cluster.yaml
   ```

2. Install CRD

   ```
   kubectl create -f https://download.elastic.co/downloads/eck/2.2.0/crds.yaml
   ```

3. Install Kubernetes Operator

   ```
   kubectl apply -f https://download.elastic.co/downloads/eck/2.2.0/operator.yaml
   ```

4. Deploy Elasticsearch Cluster

   ```
   kubectl apply -f elasticsearch.yaml
   ```

   To retrieve details about Elasticsearch cluster

   ```
   kubectl get elasticsearch
   ```

   To list the pod

   ```
   kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'
   ```

   To get the credentials
   The default username is **elastic**

   ```
   kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'
   ```

   To access using port forwarding

   ```
   kubectl port-forward service/quickstart-es-http 9200
   ```

5. Deploy Kibana

   ```
   kubectl apply -f kibana.yaml
   ```

   To retrieve details about Kibana

   ```
   kubectl get kibana
   ```

   To list the pod

   ```
   kubectl get pod --selector='kibana.k8s.elastic.co/name=quickstart'
   ```

   To access using port forwarding

   ```
   kubectl port-forward service/quickstart-kb-http 5601
   ```

   Use the same username and password as Elasticsearch

6. Filebeat on ECK

## Clean Up

## References

1. https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html
