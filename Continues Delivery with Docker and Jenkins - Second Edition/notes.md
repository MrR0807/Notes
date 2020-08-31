# Chapter 1. Introducing Continuous Delivery

# Understanding CD

Continuous Delivery is the ability to get changes of all types — including new features, configuration changes, bug fixes, and experiments—into production, or into the hands of users, safely and quickly, in a sustainable way.

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




























