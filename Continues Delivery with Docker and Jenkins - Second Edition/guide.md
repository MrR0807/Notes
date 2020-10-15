```
docker volume create jenkins-volume
```

```
docker run -p 49000:8080 -v jenkins-volume:/var/jenkins_home --name jenkins jenkins/jenkins:lts
```

For linux:
```
mkdir $HOME/jenkins_home
chown 1000 $HOME/jenkins_home
docker run -d -p 49001:8080 -v $HOME/jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:lts
```


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

Behind proxy:
```
docker run -p 49000:8080 -v jenkins-volume:/var/jenkins_home --name jenkins -e JAVA_OPTS="-Dhttp.proxyHost=host.docker.internal -Dhttp.proxyPort=3128 -Dhttps.proxyHost=host.docker.internal -Dhttps.proxyPort=3128" jenkins/jenkins:lts
```


