### Nodes list
(base) Roshans-MacBook-Pro:data roshanbharti$ k get nodes
NAME                                          STATUS   ROLES    AGE   VERSION
ip-172-31-2-53.ap-south-1.compute.internal    Ready    master   85d   v1.19.1
ip-172-31-6-190.ap-south-1.compute.internal   Ready    <none>   84d   v1.19.2
ip-172-31-9-162.ap-south-1.compute.internal   Ready    <none>   84d   v1.19.2

### Taint the node
(base) Roshans-MacBook-Pro:data roshanbharti$ k taint nodes ip-172-31-6-190.ap-south-1.compute.internal node=ip-172-31-6-190.ap-south-1.compute.internal:NoSchedule
node/ip-172-31-6-190.ap-south-1.compute.internal tainted

Example taint-toleration.yaml YAML:
```
apiVersion: mongodb.com/v1
kind: MongoDB
 
metadata:
  name: taintcheckone
  namespace: mongodb
spec:  
  opsManager:
    configMapRef:
      name: mongodb-config  
  credentials: opsmgr-credentials  
  members: 2
  persistent: true
  podSpec:
    cpu: "1"
    memory: 2G
    persistence:
      single:
        storage: 5Gi
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - ip-172-31-6-190.ap-south-1.compute.internal
    podTemplate:
      spec:
        tolerations:
          - key: "node"
            operator: "Equal"
            value: "ip-172-31-6-190.ap-south-1.compute.internal"
            effect: "NoSchedule"
  #project: taintcheckone
  type: ReplicaSet
  version: 4.2.8-ent
```

(base) Roshans-MacBook-Pro:kubeadmcluster roshanbharti$ k apply -f taint-toleration.yaml 
(base) Roshans-MacBook-Pro:kubeadmcluster roshanbharti$ k apply -f my-replica-set.yaml 

### Notice that this deployment is going to only `ip-172-31-6-190` while the other on `ip-172-31-9-162`.
```
(base) Roshans-MacBook-Pro:kubeadmcluster roshanbharti$ k get pods -o wide
NAME                                           READY   STATUS    RESTARTS   AGE     IP          NODE                                          NOMINATED NODE   READINESS GATES
mongodb-enterprise-operator-66d6977bb7-nbldd   1/1     Running   1          14d     10.47.0.1   ip-172-31-6-190.ap-south-1.compute.internal   <none>           <none>
my-replica-set-0                               1/1     Running   0          2m59s   10.32.0.4   ip-172-31-9-162.ap-south-1.compute.internal   <none>           <none>
taintcheckone-0                                1/1     Running   0          5m54s   10.47.0.3   ip-172-31-6-190.ap-south-1.compute.internal   <none>           <none>
taintcheckone-1                                1/1     Running   0          5m6s    10.47.0.5   ip-172-31-6-190.ap-south-1.compute.internal   <none>           <none>
```
