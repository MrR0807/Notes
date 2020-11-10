# The Installation Wizard
```
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

sudo apt-get update
sudo apt-get install -y openjdk-8-jre jenkins
```

# Alternate Configurations

```
sudo apt install -y docker.io
sudo usermod -aG docker cloud_user
```

Log out and then back in to apply the group change, then continue:
```
docker network create jenkins
docker volume create jenkins-docker-certs
docker volume create jenkins-data
docker container run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 docker:dind
docker container run --name jenkins-blueocean --rm --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 jenkinsci/blueocean
```

# Jenkins Installation Lab

```
sudo yum install -y java-1.8.0-openjdk-devel
sudo yum install -y wget
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum install -y jenkins
```

```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

# System Configuration

Jenkins -> Configuration.

Labels: If you'd enter "master" into the label field and select ``Usage`` as ``Only build jobs with label expressions matching this node``, it would only start jobs if the node would have label - master.

Quiet period: Wait period (seconds) after job completed.

# Setting up a Build Agent

setting up the jenkins user:

```
ls -l /var/lib
sudo mkdir /var/lib/jenkins

ls -l /var/lib | grep jenkins
sudo useradd -d /var/lib/jenkins jenkins
sudo chown jenkins:jenkins /var/lib/jenkins
```

Generating and setting ssh keys:
```
ssh-keygen

sudo mkdir /var/lib/jenkins/.ssh
cat ./.ssh/id_rsa_pub
sudo vim /var/lib/jenkins/.ssh/auhorized_keys
```

Installing java:
```
sudo apt-install openjdk-8-jre-headless
```

Getting the private key for the user:
```
cat ./.ssh/id_rsa 
```

Correcting the known hosts issue once we have ssh'd from master to agent:
```
sudo cp ./.ssh/known_hosts /var/lib/jenkins/.ssh
```

# Distributing a Jenkins Build Lab

Connect to master node -> From it, connect to slave node.
This will create ``known_hosts`` file on master.

Create jenkins folder on slave:
```
sudo mkdir /var/lib/jenkins
```

Add user in slave:
```
sudo useradd -d /var/lib/jenkins jenkins
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo mkdir /var/lib/jenkins/.ssh
```

Generate key without phrase:
```
ssh-keygen
```

```
cat ~/.ssh/id_rsa.pub
//Past into id_rsa.pub
sudo vim /var/lib/jenkins/.ssh/authorized_keys
```

Get the content of private key and use it Jenkins:
```
cat ~/.ssh/id_rsa
```

Exit into master. Make ssh folder in jenkins instalation:
```
sudo mkdir /var/lib/jenkins/.ssh
sudo cp ~/.ssh/known_hosts /var/lib/jenkins/.ssh
```
