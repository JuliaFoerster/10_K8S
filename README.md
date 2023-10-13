# K8S

#### EXERCISE 2: Deploy Mysql with 3 replicas
First of all, you want to deploy the mysql database.
Deploy Mysql database with 3 replicas and volumes for data persistence
To simplify the process you can use Helm for that.

<code>cd exercise_2</code>

##### Create a Kubernetes cluster (Minikube)

<code>brew install minikube</code>

<code>minikube start --driver docker</code>

<code>minikube status</code>

##### Use YAML files to create Volume and VolumeClaim

<code>kubectl apply -f sql-replica-pv.yaml</code>

<code>kubectl apply -f sql-replica-pvc.yaml</code>

##### Use Helm Charts to create 3 SQL Instances using peristant volumes

<code>helm repo add bitnami https://charts.bitnami.com/bitnami</code>

<code>helm search repo bitnami/</code>

<code>helm install mysql bitnami/mysql -f sql-replica.yaml</code>
