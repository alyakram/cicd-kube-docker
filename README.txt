Jenkins-CICD-docker-kubernetes
this project is about continuous delivery for docker containers
we will be continuously building docker images and deploying it to Kubernetes cluster
Scenario
- microservices Architecture of an Application
- containerized application
- continnuous code changes
so the build and release and ops team really need to cope with these regular code changes
- continuous Build & test
- Continuous Build of Container images
- regular deployment requests to Ops team

an operations team will be managing container orchestration tool like Kubernetes Cluster and they will be in charge of running your containers, so the request foes to the ops team to deploy docker images on docker

Problem
- ops team in charge of managing containers get continuous deployment requests
- manual deployment process creates dependency, the dev team is depending on the ops team for the deployment which breaks the chain of continuous delivery
- time consuming

Solution
- automate build & release process of container images
- build docker images & deploy continuously as fast as the code commits are happening
- continuous deployment

Tools Used
- kubernetes cluster which will be our container orchestration tool
- Docker engine which we will be using to build our docker images and to test them
- Jenkins will be used as a CICD server
- Docker hub where we'll be hosting our docker images
- Helm will be used for packaging and deploying our complete stack of our application to Kubernetes cluster
- git will be used as a version control system
- maven to build our java code
- sonarqube server for the code analysis

Objective
- continuous delivery for continers

Architecture Continuous delivery pipeline
when the developer makes a code change to the git repository, it will be committed to github
jenkins is going to fetch the code and this code will also include Dockerfile which will be used to build the docker image, Jenkinsfile we're going to use pipeline as a code abd also helm charts
so jenkins is going to fetch ll the changes will do its tests, we'll do the code analysis using sonarqube scanner and we'll upload the result to SonarQube server
if all the quality gates are good on the code, then we're going to build the artifact with Maven and then the docker build process will start, which will build Docker image
if everything passes then the docker image will be pushed to Dockerhub where we're ging to maintain our docker repository
if this push is successful then we're going to use helm from Jenkins, we're going to add our kops VM as a slave and we're going to run Helm from there, so jenkins will be running helm charts and will deploy helm charts to the kubernetes cluster
this helm chart deployment will create everything  for us, along with pods running through deployments, we can can also set up services, sercrets, volumes, everything through helm charts if we alreeady have them it will only implement the changes

flow of execution
1- continuous integreation setup
 a. Jenkins, SonarQube & Nexus
2- dockerhub account
3- store dockerhub credentials in Jenkins
4- setup docker engine in Jenkins
5- install Plugins in Jenkins
 a.Docker-pipeline
 b.Docker
 c.Pipeline Utility
6- Create Kubernetes cluster with kops
7- Install Helm in Kops VM
8- create Helm charts
9- Test the charts in the Kubernetes cluster in the test namespace, we'll create a namespace and we're going to test our charts in there (the entire stack is referred to as charts)
10- Add kops VM as Jenkins slave
11- create Pipeline code
12- update Git repository with
 a.Helm charts
 b.Dockerfile
 c.Jenkinsfile
13- create Jenkins job for pipeline
14- Run and test the job

We will first setup the jenkins-server ec2 instance we will set the type as t2.small
we will be able to ssh from my our IP and we will allow sonarqube to connect to jenkins for quality gate check, on port 80, and we will want to connect to it from port 8080
we will setup the SonarQube VM and will set the type ass t2.medium
SonarQube runs on port 9000, but we have Nginx service in this instance which will forward the request to sonarqube service in the sonar-setup.sh file so we can directly access on port 9000 or 80.
we will be using https://github.com/devopshydclub/vprofile-project/tree/cicd-kube we'll have the Java source code in src directory which will build from Jenkins and generate the artifact,
we have a Dockerfile, with the docker build process we are going to crate a docker image, so the artifact generated from our source code will be copied in into /usr/local/tomcat/webapps/ROOT.war location in the docker image
we have a Jenkinsfile because we're going to use PAAC in this project
we'll log into jenkins and sonarqube
generate a new sonarqube token and add it to Jenkins
allow all access from the jenkins security group in sonarqube and vice versa
we'll store the details of Dockerhub login into Jenkins credentials
we'll install docker engine in jenkins server to run Docker build commands from Jenkins
we'll add jenkins user to docker group
install docker, docker pipeline and pipeline utility steps in jenkins
we'll log into our kops ec2 instance and we'll create our kubernetes cluster
we'll create our cluster using $ kops create cluster --name=kubevpro.devops-projects.xyz --state=s3://vprofile-kops-state98 --zones=us-east-1a,us-east-1b  --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubevpro.devops-projects.xyz
which will create a cluster of 2 worker nodes and one master node all of size t2.micro and we'll store it's state in the above s3 bucket
we'll install Helm in this kops ec2 instance  which is a packaging system for definition files we can package all the definitions for our project stack and we can deploy to kubernetes cluster
so instead of managing all the definition files for deployment, services , volumes and all the different definition files we maintain and manage seperately, we can use helm instead to package all of them and run them on kubernetes cluster
we'll put the definition files we created in the helm charts dirtectory we created
and create the namespace prod that we will use from jenkins
we'll write the Jenkinsfile
we'll create a webhook from sonarqube to connect with jenkins

next stage will be deploying it to the kubernetes cluster and we are going to run the command from kops so we have to add our kops ec2 instance as a slave to Jenkins
so we'll update the security group of kops and allow ssh from Jenkins

Create a pipeline job in jenkins and choose pipeline script from git