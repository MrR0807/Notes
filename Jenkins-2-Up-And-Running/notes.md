# Chapter 1: Introducing Jenkins 2

### Pipeline

As the name implies, the Pipeline type of project is intended for creating pipelines. This is done by writing the code in the Jenkins DSL. This is the main type of project we’ll be talking about throughout the book.
As already noted, pipelines can either be written in a **“scripted” syntax style or a “declarative” syntax style**. Pipelines created in this type of project can also be made easily into Jenkinsfiles.

### Organization

Certain source control platforms provide a mechanism for grouping repositories into “organizations.” Jenkins integrations allow you to store Jenkins pipeline scripts as Jenkinsfiles in the repositories within an organization and execute based on those. Currently GitHub and Bitbucket organizations are supported, with others planned for the future. For simplicity in this book, we’ll talk mainly about GitHub Organization projects as our example.

### Summary

Jenkins 2 also provides several new project types. The Folder type allows for grouping projects together under a shared namespace and shared environment. The Multibranch Pipeline type provides easy automated job creation per branch and continuous integration, all triggered by Jenkinsfiles residing in the branches. And the organization project type extends the multibranch functionality across all projects in an organization structure on GitHub or Bitbucket.

# Chapter 2: The Foundations

Four basic areas:
* The two styles of syntax that can be used for creating pipelines
* The systems used to run the pipeline processes
* The basic structure of a pipeline
* The support environment (and tooling) that Jenkins provides for pipeline development and execution

## Syntax: Scripted Pipelines Versus Declarative Pipelines

**Scripted syntax** refers to the initial way that pipelines-as-code have been done in Jenkins. It is an imperative style, meaning it is based on defining the logic and the program flow in the pipeline script itself. It is also more dependent on the Groovy language and Groovy constructs—especially for things like error checking and dealing with exceptions.

**Declarative syntax** is a newer option in Jenkins. Pipelines coded in the declarative style are arranged in clear sections that describe (or “declare”) the states and outcomes we want in the major areas of the pipeline, rather than focusing on the logic to accomplish it.

```
// Scripted Pipeline
node('worker_node1') {
  stage('Source') { // Get code
    // get code from our Git repository
    git 'git@diyvb2:/home/git/repositories/workshop.git'
  }
  stage('Compile') { // Compile and do unit testing
    // run Gradle to execute compile and unit testing
    sh "gradle clean compileJava test"
  }
}


// Declarative Pipeline
pipeline {
  agent {label 'worker_node1'}
  stages {
    stage('Source') { // Get code
      steps {
        // get code from our Git repository
        git 'git@diyvb2:/home/git/repositories/workshop.git'
      }
    }
    stage('Compile') { // Compile and do unit testing
      steps {
        // run Gradle to execute compile and unit testing
        sh "gradle clean compileJava test"
      }
    }
  }
}
```

### Choosing Between Scripted and Declarative Syntax

In short, the declarative model should be easier to learn and maintain for new pipeline users or those wanting more ready-made functionality like the traditional Jenkins model. This comes at the price of less flexibility to do anything not supported by the structure.
The scripted model offers more flexibility. It provides the “power-user” option, allowing users to do more things with less imposed structure.

## Systems: Masters, Nodes, Agents, and Executors

In traditional Jenkins, there were only two categories: **masters** and **slaves**.

### Master

A Jenkins **master** is the primary controlling system for a Jenkins instance. It has complete access to all Jenkins configuration and options and the full list of jobs. It is the default location for executing jobs if another system is not specified.
However, it is not intended for running any heavyweight tasks. **Jobs requiring any substantial processing should be run on a system other than the master.**

### Node

**Node** is the generic term that is used in Jenkins 2 to mean any system that can run Jenkins jobs. This covers both masters and agents, and is sometimes used in place of those terms. Furthermore, a node might be a container, such as one for Docker.

### Agent

An **agent** is the same as what earlier versions of Jenkins referred to as a **slave**. Traditionally in Jenkins, this refers to any nonmaster system. The idea is that these systems are managed by the master system and allocated as needed, or as specified, to handle processing the individual jobs.

#### Directives Versus Steps

**node** is associated with a Scripted Pipeline.

```
// Scripted Pipeline
node('worker') {
  stage('Source') { // Get code
    // Get code from our Git repository
```

**agent**, on the other hand, is a directive in a Declarative Pipeline.
```
// Declarative Pipeline
pipeline {
  agent {label:'worker'}
  stages {
    stage('Source') { // Get code
```

Just use **node** for Scripted Pipelines and **agent** for Declarative Pipelines.

### Executor

**executor** is just a slot in which to run a job on a node/agent. A node can have zero or more executors. **The number of executors defines how many concurrent jobs can be run on that node.** When the master funnels jobs to a particular node, there must be an available executor slot in order for the job to be processed immediately. Otherwise, it will wait until an executor becomes available.
















































