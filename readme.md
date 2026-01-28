Deploying MERN stack application in aws kubernetes cluster - End-to-End-Kubernetes-Three-tier-Devsecops Project:

The project is mainly divided into 3 stages 



Creating Jenkins server:
We will create a jenkins server through which provision AWS-EKS cluster using terraform , can be triggered using jenkins pipeline

EC2 instance specification:
Application and OS Image : Ubuntu 22.04
Instance type: t3.xlarge
Keypair : Proceed witout keypair


<img width="1235" height="853" alt="Screenshot 2026-01-28 at 11 36 19 am" src="https://github.com/user-attachments/assets/1e3072e2-80d9-4990-aa23-ecb465a4c486" />





<img width="1115" height="407" alt="Screenshot 2026-01-28 at 11 36 57 am" src="https://github.com/user-attachments/assets/8759a5f6-958a-42f1-b05d-c89417d05901" />

Opening ports for jenkins and sonarqube : 8080 & 9000


<img width="1111" height="685" alt="Screenshot 2026-01-28 at 11 40 38 am" src="https://github.com/user-attachments/assets/c1c71f52-ef48-4e98-b535-6a04f825f439" />

configuring storage to 30gb and adding instance profile 

<img width="1091" height="604" alt="Screenshot 2026-01-28 at 11 43 27 am" src="https://github.com/user-attachments/assets/510a2582-ff7d-4b6a-8288-0576726ba8df" />

updating user data : automating installation of java , jenkins , docker , aws-cli, kubectl , eksctl , terraform , trivy , helm

#!/bin/bash
# For Ubuntu 22.04
# Intsalling Java
sudo apt update -y
sudo apt install openjdk-17-jre -y
sudo apt install openjdk-17-jdk -y
java --version

# Installing Jenkins
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \ https://pkg.jenkins.io/debian/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \ https://pkg.jenkins.io/debian binary/ | sudo tee \  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y

# Installing Docker 
#!/bin/bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock

# If you don't want to install Jenkins, you can create a container of Jenkins
# docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-container jenkins/jenkins:lts

# Run Docker Container of Sonarqube
#!/bin/bash
docker run -d  --name sonar -p 9000:9000 sonarqube:lts-community


# Installing AWS CLI
#!/bin/bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install

# Installing Kubectl
#!/bin/bash
sudo apt update
sudo apt install curl -y
sudo curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client


# Installing eksctl
#! /bin/bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Installing Terraform
#!/bin/bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y

# Installing Trivy
#!/bin/bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy -y


# Intalling Helm
#! /bin/bash
sudo snap install helm --classic
