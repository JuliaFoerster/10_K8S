# K8S

<details>
<summary> EXERCISE 1: Create a Kubernetes cluster.
</summary>
  <br>
  <code>brew install minikube</code> <br>
  <code>minikube start --driver docker</code> <br>
  <code>minikube status</code> 
<br>

</details>

<details>
<summary> EXERCISE 2: Deploy Mysql with 3 replicas.
</summary>
  <br>
  First of all, you want to deploy the mysql database. Deploy Mysql database with 3 replicas and volumes for data persistence. To simplify the process you can use Helm for that.<br>
<br>
  
##### Execute deployment from directory exercise_2/3_bitnami_with_replia:
<code> cd exercise_2/3_bitnami_with_replia</code>

##### 1. Use YAML files to create Volume and VolumeClaim

<code>kubectl apply -f sql-replica-pv.yaml</code> <br>
<code>kubectl apply -f sql-replica-pvc.yaml</code> <br>

##### 2. Use Helm Charts to create 3 SQL Instances using peristant volumes

<code>helm repo add bitnami https://charts.bitnami.com/bitnami</code> <br>
<code>helm search repo bitnami/</code> <br>
<code>helm install mysql bitnami/mysql -f sql-replica.yaml</code> <br>

##### 3. Check if pods run as expected: 
##### Enter pod
<code> kubectl exec -it pod/mysql-primary-0 -- /bin/bash </code>

##### DB login 
<code> mysql -u <MYSQL_USERNAME> -p <MYSQL_PASSWORD</code> <br>
-> <code> mysql -u dev_user -p passworddev</code><br>
</details>


<details>
<summary> EXERCISE 3: Deploy your Java Application with 3 replicas.
</summary>
<br>
  Deploy your Java application with 3 replicas.
With docker-compose, you were setting env_vars on server. In K8s there are own components for that, so create ConfigMap and Secret with the values and reference them in the application deployment config file.<br>

##### Execute deployment from directory k8s

##### Create docker image from Java App and push it into your DockerHub Repository. 

<code>cd docker-exercise</code><br>
<code>gradle build</code><br>

Rename generated jar file to: java-mysql-project-1.0-SNAPSHOT.jar <br>

- Create and tag a docker image using existing Dockerfile:
<code>docker build -t <DOCKERHUB_USERNAME>/java_app:1.0 .</code><br>
- Check if the image got build:
<code>docker images</code><br>
- Login to Docker Hub:
<code>docker login</code><br>
- Push Image into your Dockerhub Repository:
<code>docker push <DOCKERHUB_USERNAME>/java-app:1.0</code><br>

##### 1. Create Key (login to Registry and create Secret in K8S)

```
DOCKER_REGISTRY_SERVER=https://index.docker.io/v1/
DOCKER_USER=your docker username
DOCKER_EMAIL=your dockerhub email
DOCKER_PASSWORD= dockerhub pwd

kubectl create secret docker-registry my-registry-key1 \
--docker-server=<DOCKER_REGISTRY_SERVER>
--docker-username=<DOCKERHUB_USERNAME> \
--docker-password=<DOCKERHUB_PASSWORD> \
--docker-email=<DOCKERHUB_EMAIL>
```

###### 2. Execute following commands
<code>kubectl apply -f mysql-secret.yaml</code><br>
<code>kubectl apply -f mysql-configmap.yaml</code><br>
<code>kubectl apply -f deployment.yaml</code><br>
</details>

<details>
<summary> EXERCISE 4: Deploy phpmyadmin.
</summary>
  <br>
  
  ###### 1. Create deployment yaml file for phpmyadmin instance and deploy:
  <code>kubectl apply -f phpmyadmin.yaml</code><br>
</details>

<details>
<summary> EXERCISE 5: Deploy ingress controller
</summary>
  <br>
  
  We want users to access the application using the IP address and instead use a domain name -> Install Ingress controller in 
  the cluster and configure ingress access for your application.
  
  ###### Create Controller 
  <code>minikube addons enable ingress </code><br>

</details>

<details>
<summary> EXERCISE 6: Create Ingress rule.
</summary>
  <br>

  ###### 1. Write Routing Rule and apply it to cluster
  <code>kubectl apply -f kubectl apply -f java-app-ingress.yaml</code><br>
  <br>
  This will throw an error: <br>
  Error from server (InternalError): error when creating "java-app-ingress.yaml": Internal error occurred: failed calling webhook  "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "[...]": dial tcp 10.111.212.254:443: connect: connection refused <br>

  As described here: https://medium.com/ci-cd-devops/internal-error-occurred-failed-calling-webhook-validate-nginx-ingress-kubernetes-io-b5008e628e03 
  you can solve this error with: <br>
   <br>
  <code>kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission</code><br>
  
  ###### 2. Starting minikube tunnel
  Everything coming from localhost 127.0.0.1  will be redirected to minikube ip
  
  <code>sudo minikube tunnel</code><br>

  ###### 3. Ensure your /etc/hosts file contains the correct entry:
  map "my-app.com" to "127.0.0.1." <br>

  ###### 4. Check if App availabe via Browser:
  1) etc/hosts file will redirect requests from http://my-app.com/ to 127.0.0.1<br>
  2) minikube tunnel redirects request from 127.0.0.1 to minikube (services).<br>
       E.g.<br>
         - phpmyadmin service available on http://my-app.com:8081/<br>
         - http://my-app.com/ will show team member roles (a picture)<br>
         - http://my-app.com/get-data shows list of existing team members<br>

</details>


<details>
<summary> EXERCISE 7: Port-forward for phpmyadmin
</summary>
  <br>

This exercise does not make much sense in our scenario because our app is running on minikube/localhost (see exercise 6).

phpMyAdmin Security Concerns: phpMyAdmin is a tool for managing MySQL databases, and it can potentially have security implications if it's exposed to the public internet. By default, it might have vulnerabilities or could be a target for attacks. So, to mitigate these concerns, you choose not to expose it to the public.<br>

Port Forwarding: Instead of exposing phpMyAdmin directly via my-app.com/8081, you configure port forwarding. Port forwarding allows you to access a specific service running inside your Kubernetes cluster from your local machine (localhost) without exposing it to the public internet. This means that phpMyAdmin remains hidden within your cluster, and you can access it securely when needed.<br>

The kubectl port-forward command is used to create a network tunnel between your local machine and a service running inside a Kubernetes cluster: <br>

<code> kubectl port-forward svc/phpmyadmin-service 8081:8081</code><br>

Now phpmyadmin is not available on my-app.com/8081 anymore but on localhost/8081 (which is the same for us, as mentioned at the beginning of this exercise).


</details>

