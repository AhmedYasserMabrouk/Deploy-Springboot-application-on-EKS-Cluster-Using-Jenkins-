# Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins

<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/683a4cc1-5235-48b9-8d9e-16dbf5d8a7c2">

<br>

## Create Jenkins-Ec2 Instance

#### Change Host Name to Jenkins

```
sudo hostname jenkins
```
#### Perform update

```
sudo apt update
```



#### Jenkins Setup
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

```

<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/3b6c06a9-41db-4994-8f87-9e9c55b20677">



### Install Kubectl:

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.11/2023-03-17/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo cp ./kubectl /usr/local/bin
export PATH=/usr/local/bin:$PATH
```

### Install AWScli:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
sudo apt install unzip
sudo unzip awscliv2.zip  
sudo ./aws/install
aws --version
```
### Install EKSctl:

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

```



#### Maven Installation

```
sudo apt install maven -y
```
#### Add jenkins user to Docker group

```
sudo usermod -a -G docker jenkins
```

#### Restart Jenkins service
```
sudo service jenkins restart
```

#### Reload system daemon files
```
sudo systemctl daemon-reload
```

#### Restart Docker service

```
sudo service docker stop
sudo service docker start

```
<br>

#### Install this plugins on Jenkins
<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/bcd25cbe-3fb5-496f-ab85-a5e725bceda6">


#### Create ECR Registry
<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/2aa99d1c-08c5-496e-9734-f8b67ee0a239">














##### Switch to Jenkins user

```
sudo su jenkins
```

#### Create EKS Cluster with two worker nodes using eksctl
```
eksctl create cluster --name MY-Cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --managed --nodes 2 
```

#### you can view the kubeconfig file by entering the below command and Create Credentials for connecting to Kubernetes Cluster using kubeconfig in Jenkins

```
cat  /var/lib/jenkins/.kube/config

```

#### Create a pipeline in Jenkins and Copy the pipeline code from below
```
pipeline {
    agent any
    
    tools{
        maven 'Maven'
    }
    
    environment {
        registry = "851725182873.dkr.ecr.us-east-1.amazonaws.com/myecr-repo"
    }

    stages {
        stage('CheckOut') {
            // github repo
            steps {
                
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-.git']])
            }
        }
        
         stage('Build Jar') {
            steps {
                sh 'mvn clean install'
            }
        }
        
         // Build Docker images
         stage('Building image') {
      steps{
           script {
             dockerImage = docker.build registry 
        }
      }
    }
     // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
             
             // login to ECR
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 851725182873.dkr.ecr.us-east-1.amazonaws.com'
                
                //push docker image to ECR
                sh 'docker push 851725182873.dkr.ecr.us-east-1.amazonaws.com/myecr-repo:latest'
         }
        }
      }
      
      
    stage('Kubernetes Deploy') {
        steps{   
           script {
                //configfile
               withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes-secret', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                // Run yaml file on EKS Cluster
               sh ('kubectl apply -f  eks-deploy-k8s.yaml')
               }
            }
    
    
        
        
    }
}
}
}

```


```
kubectl get svc
```

<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/3bb96378-7739-45e4-b18f-a0f12351ab7c">



## Running Application

<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/58373a9d-6653-4d1e-b7c2-cb3210d39f77">

<img src="https://github.com/AhmedYasserMabrouk/Deploy-Springboot-application-on-EKS-Cluster-Using-Jenkins-/assets/166556717/892815fd-17a5-46f1-bf58-ef49a5bb39ac">










