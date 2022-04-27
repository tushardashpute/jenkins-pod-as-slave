# jenkins-pod-as-slave
jenkins-pod-as-slave


Install Jenkins:

docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts-jdk11

Install Sonar:

docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube 

sonar-token :

027aff09fd16f65e23a1b0ab7cde09d2e3ca43e9

jenkins-sonar-webhooks

http://http://3.133.135.124:8080/sonarqube-webhook/

Configure Pod as slave:


jenkins/jnlp-slave
