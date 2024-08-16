## Applications and prerequisites 

- java 
- Jenkins
- Maven
- SonarQube
- Docker
- Docker Hub
- Shell script
- minikube
- Argo CD

## Installation on AWS EC2 Instance

- Go to AWS Console
- Instances(running)
- Launch instances
- add inbound traffic rule

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">



### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins

Install Java

```
sudo apt update
sudo apt install openjdk-17-jre
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```


### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      

### Click on Install suggested plugins


Wait for the Jenkins to Install suggested plugins


Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]


Jenkins Installation is Successful. You can now starting using the Jenkins 

## Install the Docker Pipeline plugin in Jenkins:

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab,search for "Docker Pipeline" and "SonarQube Scanner".
   - Select the plugin and click the Install button.
   - Restart Jenkins after the plugin is installed.
   

Wait for the Jenkins to be restarted.


## Next Steps

### Configure a Sonar Server locally

```
sudo su -
apt install unzip
adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Hurray !! Now you can access the SonarQube Server on `http://<ip-address>:9000` 

```
sonarqube Login: admin
sonarqube pass: admin
```

## Create a pipeline and add token and credentials.

   - add Git repo in the pipeline and add Git credentials with secret text and set ID as "github"
   - add Sonarqube credentials with secret text and set ID as "sonarqube"
   - add Dockerhub credentials with username & pass and set ID as "docker-cred"
     

## Docker Slave Configuration

Run the below command to Install Docker

```
sudo su -
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

## Change the old sonarqube url with latest public IP in JenkinsFile
```
"http://<ec2-instance-public-ip>:9000"
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

## Argo CD

To start the argoCD

```
minikube start

Creating a name space 

kubectl create namespace argocd4

kubectl apply -n argocd4 -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl port-forward svc/argocd-server -n argocd4 8080:443
```
open the new tab and enter the below command
```
kubectl -n argocd4 get secret argocd-initial-admin-secret -o yaml

copy the file something look like this and give the command like

echo dXNlcjpwYXNzd29yZA== | base64 --decode

```

Argo CD user name: admin
pass: copy from the terminal.




==============================================================================

# Spring Boot based Java web application (Local host)

This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

## Execute the application locally and access it using your browser

Checkout the repo and move to the directory

```
git clone https://github.com/Harsha640/Jenkins-CICD-practice
///cd /Users/ram/Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/spring-boot-app///
```
install in the spring-boot-app path
Execute the Maven targets to generate the artifacts

```
mvn clean package
```

### The Docker way

Build the Docker Image

Remove the old base image and add the below image in visual studio code.

```
FROM eclipse-temurin:21 
```


Ready to build the image
```
docker build -t ultimate-cicd-pipeline:v1 .
```

```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Hurray !! Access the application on `http://<ip-address>:8010`

