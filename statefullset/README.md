# Statefullset Demo

Statefullset demo using Cassandra

# Steps

1. Create EKS Cluster

   ```
   eksctl create cluster -f eks-cluster.yaml
   ```

2. Create headless service

   ```
   kubectl apply -f cassandta-service.yaml
   ```

3. Create statefullset

   ```
   kubectl apply -f cassandra-statefulset.yaml
   ```

4. Check

   - Get cassandta statefullset

   ```
   kubectl get statefulset cassandra
   ```

   - Run cassandra nodetool inside pod to display status of the ring

   ```
   kubectl exec -it cassandra-0 -- nodetool status
   ```

   - Change replicas to 4

   ```
   kubectl edit statefulset cassandra
   ```

5. Cleanup

   ```
   kubectl delete -f cassandra-statefulset.yaml
   kubectl delete -f cassandra-service.yaml

   eksctl delete cluster -f eks-cluster.yaml
   ```

# References

1. https://kubernetes.io/docs/tutorials/stateful-application/cassandra/
