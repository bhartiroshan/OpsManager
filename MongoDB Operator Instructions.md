# Scenario 01: Ops Manager running outise Kubernetes

# Kubernetes Operator Install

1. Clone the MongoDB Kubernetes Operator git repository
   a. git clone https://github.com/mongodb/mongodb-enterprise-kubernetes.git :

2. Install the CRDs(Custom Resource Definitions), it allows Kubernetes to understand Custome Objects like MongoDB OpsManager
   a. kubectl create namespace mongodb :
   b. kubectl apply -f mongodb-enterprise-kubernetes/crds.yaml :
   c. kubectl get crd : # Verify CRDs are installed

3. Install the Operator
   a. kubectl apply -f mongodb-enterprise-kubernetes/mongodb-enterprise.yaml :
   b. kubectl get pods : # verify Operator pod is running 

4. Create a Config Map
   a. Create a config Map YAML Specification, see a sample one below:

    {
    ---
    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: kube-config ------------------------------------------------------------------------------|
        namespace: mongodb                                                                              |
    data:                                                                                               |
        projectName: kubeorg # Project Name that gets Created when deployment MongoDB Resource          |
        orgId: 5f2c40ea25825d0071e6adbd                                                                 |
        baseUrl: http://ec2-13-234-204-184.ap-south-1.compute.amazonaws.com:81 # Ops Manager URL        |
    }                                                                                                   |
                                                                                                        |
   b. kubectl apply -f kube-config.yaml :                                                               |
   c. kubectl get configmap :                                                                           |
                                                                                                        |
5. Create API Secret                                                                                    |
   a. Sample Secret file :                                                                              |
                                                                                                        |
    {                                                                                                   |
        ---                                                                                             |
        apiVersion: v1                                              |-----------------------------------|
        kind: Secret                                                |
        metadata:                                                   |
            name: opsmgr-credentials  ------------------------------|
            namespace: mongodb                                      |
        stringData:                                                 |
            user: QHZFJYXE                                          |
            publicApiKey: 474cf6ae-e915-4b7e-bde9-3f7c72e91211      |
    }                                                               |
                                                                    |
   b. kubectl apply -f opsmgr-credentials.yaml :                    |
                                                                    |
6. Deploy a MongoDB Replica-Set                                     |
   a. Sample MongoDB replica-Set :                                  |
                                                                    |
    {                                                               |
        ---                                                         |
        apiVersion: mongodb.com/v1                                  |
        kind: MongoDB                                               |
        metadata:                                                   |
          name: my-replica                                          |
        spec:                                                       |
          members: 1                                                |
          #Test Comment                                             |
          version: 4.2.1-ent                                        |
          opsManager:                                               |
            configMapRef:                                           |
            name: kube-config --------------------------------------|
          credentials: opsmgr-credentials --------------------------|
          type: ReplicaSet
          persistent: true   
    }

   b. kubectl apply -f my-replica.yaml :
   c. kubectl get mdb : # To view the MongoDB Objects 
   d. kubectl get pods : # To view the replica-set Pods        


# Scenario 02: Installing OM in Kubernetes

1. Repeat Step 1-3(see above) to install Operator :

2. Create Ops Manager Admin Secret, first user login :
    a. kubectl create secret generic ops-manager-admin-secret \ <-------------------------------------------|
        --from-literal=Username="roshan.bharti@mongodb.com" \                                               |
        --from-literal=Password="Mongodb@123" \                                                             |
        --from-literal=FirstName="Roshan" \                                                                 |
        --from-literal=LastName="Bharti"                                                                    |
                                                                                                            |
                                                                                                            |
2. Deploy Ops Manager resource :                                                                            |
    b. See a sample specification file                                                                      |
    {                                                                                                       |
        ---                                                                                                 |    
        apiVersion: mongodb.com/v1                                                                          |
        kind: MongoDBOpsManager                                                                             |
        metadata:                                                                                           |
            name: ops-manager                                                                               |
        spec:                                                                                               |
            # the number of Ops Manager instances to run. Set to value bigger                               |
            # than 1 to get high availability and upgrades without downtime                                 |
            replicas: 3                                                                                     |
                                                                                                            |
            # the version of Ops Manager distro to use                                                      |
            version: 4.4.0                                                                                  |
                                                                                                            |
            # the name of the secret containing admin user credentials.                                     |    
            # Either remove the secret or change the password using Ops Manager UI after the Ops Manager    |
            # resource is created!                                                                          |
            adminCredentials: ops-manager-admin-secret # <--------------------------------------------------|
        
            # the application database backing Ops Manager. Replica Set is the only supported type
            # Application database has the SCRAM-SHA authentication mode always enabled
            applicationDatabase:
            members: 3
            # optional. Configures the version of MongoDB used as an application database.
            # The bundled MongoDB binary will be used if omitted and no download from the Internet will happen
            version: 4.2.6-ent
            persistent: true
    }

    b. kubectl apply -f ops-manager.yaml 
    c. kubectl get om # View the OM resource
    d. kubectl get pods # See the OM Pod status

    
        