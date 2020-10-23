# Chapter 1. Introducing Continuous Delivery

# Understanding CD

Continuous Delivery is the ability to get changes of all types — including new features, configuration changes, bug fixes, and experiments — into production, or into the hands of users, safely and quickly, in a sustainable way.

**How long would it take your organization to deploy a change that involves just a single line of code? Do you do this on a repeatable, reliable basis?**

# The automated deployment pipeline

## Continuous Integration (CI)

The CI phase provides the first feedback to developers. **It checks out the code from the repository, compiles it, runs unit tests, and verifies the code quality.** If any step fails, the pipeline execution is stopped and the first thing the developers should do is fix the CI build. The essential aspect of this phase is time.

### The testing matrix

* **Acceptance Testing (automated):** These are tests that represent functional requirements seen from the business perspective. They are written in the form of stories or examples by clients and developers to agree on how the software should work.
* **Unit Testing (automated):** These are tests that help developers to provide high-quality software and minimize the number of bugs.
* **Exploratory Testing (manual):** This is the manual black-box testing, which tries to break or improve the system.
* **Non-functional Testing (automated):** These are tests that represent system properties related to performance, scalability, security, and so on.

**DevOps culture means, in a sense, coming back to the roots. A single person or a team is responsible for all three areas: Development, Quality Assurance, Operations.**

**Continuous Delivery** is not the same as **Continuous Deployment**. The latter means that **each commit to the repository is automatically released to production.** **Continuous Delivery** is less strict and means that each **commit ends up with a release candidate, so it allows the last step (release to production) to be manual.**

# Chapter 3. Configuring Jenkins

# What is Jenkins?

It is the most popular tool for implementing Continuous Integration and Continuous Delivery processes.

Jenkins has a built-in mechanism for the master/slave mode, which distributes its execution across multiple nodes, located on multiple machines.

# Installing Jenkins

```
docker run -p <host_port>:8080 -v <host_volume>:/var/jenkins_home jenkins/jenkins:2.150.3
```

## Initial configuration

* Open Jenkins in the browser, at ``http://localhost:<host_port>`` (for binary installations, the default port is 8080).
* Jenkins should ask for the administrator password. It can be found in the Jenkins logs:

```
docker logs jenkins

...
Jenkins initial setup is required. An admin user has been 
created 
and a password generated.
Please use the following password to proceed to installation:

c50508effc6843a1a7b06f6491ed0ca6
...
```

## Jenkins Hello World

Click on New Item (top left corner) -> Pipeline.

```
pipeline {
     agent any
     stages {
          stage("Hello") {
               steps {
                    echo 'Hello World'
               }
          }
     }
}
```

# Jenkins architecture

## Master and slaves

Jenkins becomes overloaded sooner than it seems. Even in the case of a small (micro) service, the build can take a few minutes. That means that one team committing frequently can easily kill the Jenkins instance.

For that reason, unless the project is really small, Jenkins should not execute builds at all, but delegate them to the slave (agent) instances. To be precise, the Jenkins we're currently running is called the **Jenkins master**, and it can delegate execution tasks to the **Jenkins agents**.

![Jenkins-Master-Slave.PNG](pictures/Jenkins-Master-Slave.PNG)

In a distributed builds environment, the Jenkins master is responsible for the following:

* Receiving build triggers (for example, after a commit to GitHub)
* Sending notifications (for example, email or HipChat messages sent after a build failure)
* Handling HTTP requests (interaction with clients)
* Managing the build environment (orchestrating the job executions on slaves)

Agents should also be as generic as possible.

## Scalability

**Vertical scaling** means that when the master's load grows, more resources are applied to the master's machine. So, when new projects appear in our organization, we buy more RAM, add CPU cores, and extend the HDD drives.

**Horizontal scaling** means that when an organization grows, more master instances are launched. This requires a smart allocation of instances to teams, and, in extreme cases, each team can have its own Jenkins master. In that case, it might even happen that no slaves are needed.

## Test and production instances

It means there should always be two instances of the same Jenkins infrastructure: test and production. The test environment should always be as similar as possible to the production, so it requires a similar number of agents attached.

# Configuring agents

How do we set up an agent and let it communicate with the master?

## Communication protocols

In order for the master and the agent to communicate, the bi-directional connection has to be established. There are different options for how it can be initiated:
* **SSH**. The master connects to the slave using the standard SSH protocol. This is the most convenient and stable method because it uses standard Unix mechanisms.
* **Java web start**. A Java application is started on each agent machine and the TCP connection is established between the Jenkins slave application and the master Java application. This method is often used if the agents are inside the fire-walled network and the master cannot initiate the connection.

## Setting agents

At the low level, agents always communicate with the Jenkins master using one of the protocols described previously. However, at the higher level, we can attach slaves to the master in various ways. The differences concern two aspects:
* **Static versus dynamic**. The simplest option is to add slaves permanently in the Jenkins master. The drawback of such a solution is that we always need to manually change something if we need more (or fewer) slave nodes. A better option is to dynamically provision slaves as they are needed.
* **Specific versus general-purpose**. Agents can be specific (for example, different agents for the projects based on Java 7 and Java 8) or general-purpose (an agent acts as a Docker host and a pipeline is built inside a Docker container).

These differences resulted in four common strategies for how agents are configured:
* **Permanent agents**
* **Permanent Docker agents**
* **Jenkins Swarm agents**
* **Dynamically provisioned Docker agents**

### Configuring permanent agents

``Manage Jenkins`` -> ``Manage Nodes`` -> ``New Node``.

![permanent-agent.png](pictures/permanent-agent.png)

* ``Name``: This is the unique name of the agent
* ``Description``: This is an human-readable description of the agent
* ``# of executors``: This is the number of concurrent builds that can be run on the slave
* ``Remote root directory``: This is the dedicated directory on the slave machine that the agent can use to run build jobs (for example, /var/jenkins); the most important data is transferred back to the master, so the directory is not critical
* ``Labels``: This includes the tags to match the specific builds (tagged the same); for example, only projects based on Java 8
* ``Usage``: This is the option to decide whether the agent should only be used for matched labels (for example, only for Acceptance Testing builds), or for any builds
* ``Launch method``: This includes the following:
  * ``Launch agent via Java Web Start``: Here, the connection will be established by the agent; it is possible to download the JAR file and the instructions on how to run it on the slave machine
  * ``Launch agent via execution of command on the master``: This is the custom command run on the master to start the slave; in most cases, it will send the Java Web Start JAR application and start it on the slave (for example, ssh <slave_hostname> java -jar ~/bin/slave.jar)
  * ``Launch slave agents via SSH``: Here, the master will connect to the slave using the SSH protocol
* ``Availability``: This is the option to decide whether the agent should be up all the time or the master should turn it offline under certain conditions

**!NOTE**. The Java Web Start agent uses port 50000 for communication with Jenkins Master; therefore, if you use the Docker-based Jenkins master, you need to publish that port (-p 50000:50000).

When the agents are set up correctly, **it's possible to update the master node configuration with ``# of executors`` set to 0**, so that no builds will be executed on it and it will only serve as the Jenkins UI and the builds' coordinator.

As we've already mentioned, the drawback of such a solution is that we need to maintain multiple slave types (labels) for different project types. In our example, if we have three types of projects (java7, java8, and ruby), then we need to maintain three separately labeled (sets of) slaves.

### Permanent Docker agents

The idea behind this solution is to permanently add general-purpose slaves. Each slave is identically configured (with Docker Engine installed), and each build is defined along with the Docker image inside which the build is run.

#### Configuring permanent Docker agents

The configuration is static, so it's done exactly the same way as we did for the permanent agents. The only difference is that we need to install Docker on each machine that will be used as a slave. After the slaves are configured, we define the Docker image in each pipeline script:
```
pipeline {
     agent {
          docker {
               image 'openjdk:8-jdk-alpine'
          }
     }
     ...
}
```

When the build is started, the Jenkins slave starts a container from the Docker image, ``openjdk:8-jdk-alpine``, and then executes all the pipeline steps inside that container.

![permanent-docker-slaves.png](pictures/permanent-docker-slaves.png)

### Jenkins Swarm agents

#### Configuring Jenkins Swarm agents

The first step to using Jenkins Swarm is to install the Self-Organizing Swarm Plug-in Modules plugin in Jenkins. We can do it through the Jenkins web UI, under ``Manage Jenkins`` and ``Manage Plugins``.

The second step is to run the Jenkins Swarm slave application on every machine that would act as a Jenkins slave. We can do it using the ``swarm-client.jar`` application.

**!NOTE**. The ``swarm-client.jar`` application can be downloaded from the Jenkins Swarm plugin page, at https://wiki.jenkins.io/display/JENKINS/Swarm+Plugin. On that page, you can also find all the possible options of its execution.

In order to attach the Jenkins Swarm slave node, it's enough to run the following command:
```
$ java -jar swarm-client.jar -master <jenkins_master_url> -username <jenkins_master_user> -password <jenkins_master_password> -name jenkins-swarm-slave-1
```

After successful execution, we should notice that a new slave has appeared on the Jenkins master, as presented in the following screenshot:

![added-jenkins-swarm-agent-slave.png](pictures/added-jenkins-swarm-agent-slave.png)

**!NOTE**. The other possibility to add the Jenkins Swarm agent is to use the Docker image built from the ``swarm-client.jar`` tool.

#### Understanding Jenkins Swarm agents

At first glance, Jenkins Swarm may not seem very useful. After all, we have moved setting agents from the master to the slave, but we still have to do it manually. However, apparently, with the use of a clustering system such as Kubernetes or Docker Swarm, Jenkins Swarm enables the dynamic scaling of slaves on a cluster of servers.

### Dynamically provisioned Docker agents

Another option is to set up Jenkins to dynamically create a new agent each time a build is started. Such a solution is obviously the most flexible one, since the number of slaves dynamically adjusts to the number of builds.

#### Configuring dynamically provisioned Docker agents

First, we need to install the ``Docker plugin``. Configuration steps:
* Open the ``Manage Jenkins`` page.
* Click on the ``Configure System`` link.
* At the bottom of the page, there is the ``Cloud`` section.
* Click on ``Add a new cloud`` and choose ``Docker``.
* Fill in the Docker agent details, as shown in the following screenshot.
* Most parameters do not need to be changed; however (apart from selecting Enabled), we need to at least set the Docker host URL (the address of the Docker host machine where agents will be run).
* Click on Docker Agent templates... and select Add Docker Template.
* Fill in the details about the Docker slave image.

![dynamic-docker-slave.png](pictures/dynamic-docker-slave.png)

**!NOTE**. If you plan to use the same Docker host where the master is running, then the Docker daemon needs to listen on the ``docker0`` network interface. You can do it in a similar way as to what's described in the Installing on a server section ofChapter 2, Introducing Docker, by changing one line in the ``/lib/systemd/system/docker.service`` file to ``ExecStart=/usr/bin/dockerd -H 0.0.0.0:2375 -H fd://``.

![docker-agent-template.png](pictures/docker-agent-template.png)


* ``Docker Image``: The most popular slave image from the Jenkins community is evarga/jenkins-slave
* ``Instance Capacity``: This defines the maximum number of agents running at the same time; for the beginning, it can be set to 10

#### Understanding dynamically provisioned Docker agents

Dynamically provisioned Docker agents can be treated as a layer over the standard agent mechanism. It changes neither the communication protocol nor how the agent is created. So, what does Jenkins do with the Docker agent configuration we provided?

![master-slave-dynamic-docker-overview.png](pictures/master-slave-dynamic-docker-overview.png)











































