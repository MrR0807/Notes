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
























