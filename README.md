# jenkins-container
Build Jenkins Container with Docker and Terraform inside
Create an EC2 instance (server) on AWS platform
ssh into the server
Create a directory called Jenkins (mkdir Jenkins)
cd into the Jenkins directory (cd Jenkins)
create a Dockerfile (vi Dockerfile)
Input the code below into the Dockerfile 

FROM jenkins/jenkins:lts
USER root
RUN apt-get update -y && apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
RUN curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey
RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
RUN apt-get update -y
RUN apt-get install -y docker-ce docker-ce-cli containerd.io
RUN curl -O https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip && unzip terraform_0.11.13_linux_amd64.zip -d /usr/local/bin/
USER ${user}

Run docker build command (docker build -t jenkins:terraform .)
list your docker images (docker image ls)
Then:
create a terraform file (touch main.tf)
Then vi main.tf
Input the code below into the editor, making sure you add the amazon provier with access keys and availability zone
# Jenkins Volume
resource "docker_volume" "jenkins_volume" {
  name = "jenkins_data"
}

# Start the Jenkins Container
resource "docker_container" "jenkins_container" {
  name  = "jenkins"
  image = "jenkins:terraform"
  ports {
    internal = "8080"
    external = "8080"
  }

  volumes {
    volume_name    = "${docker_volume.jenkins_volume.name}"
    container_path = "/var/jenkins_home"
  }

  volumes {
    host_path      = "/var/run/docker.sock"
    container_path = "/var/run/docker.sock"
  }
}

Then:
Initialize terraform (terraform init)
Plan the deployment (terraform plan -out=tfplan)
Deploy Jenkins (terraform apply tfplan)
Get admin password (docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword)
Then:

Add port 8080 to security group on aws ec2 instance
Open browser, input http://serverpublicip:8080 to log into Jenkins as administrator
