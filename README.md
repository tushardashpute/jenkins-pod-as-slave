# jenkins-pod-as-slave

Steps :
1. Install Jenkins and do initital setup
2. Install Docker Pipeline plugin /sonarqube plugin and kubenetes plugin on jenkins
3. Install sonarqube server
 - Generate sonar token
 - Create webhook for jenkins
4. Generate kubeconfig to connect jenkins with EKs(for creating POD's)
5. Create jenkins job using jenkinsfile.

**1. Install Jenkins and do initital setup**

        sudo yum install java-1.8.0
        sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
        sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
        sudo yum install jenkins -y
        service jenkins status
        service jenkins start
        service jenkins status

        Now you can access the Jenkins on "http://host_public_ip:8080" 
        password will be stored at : cat /var/lib/jenkins/secrets/initialAdminPassword
        
**2. Install Docker Pipeline plugin /sonarqube plugin/kubernetes on jenkins**

        Goto manage jenkins --> manage plugins --> available and install Docker pipeline plugin and sonarqube plugin.

**3. Install sonarqube server**

  [Sonarqube installaton](./sonarqube-installation.md) 
    
    Follow this link for sonarqube server installation. Once installation is done need to do below steps:

        Integrate Sonarqube with jenkins
        1. Add sonarqube plugin 
        2. Add SonarQube servers with in jenkins
- Need to create authentication token with in Sonarqube server

  ![image](https://user-images.githubusercontent.com/68885738/90910319-bebffd00-e3f4-11ea-8590-c9ae9018973e.png)

- Need to create webhook with in Sonarqube server (use Jenkins server URL)

 ![image](https://user-images.githubusercontent.com/68885738/90953421-06906400-e489-11ea-9f1d-859b3b9fa7b8.png)

  Click on My Account

  ![image](https://user-images.githubusercontent.com/68885738/90910508-0ba3d380-e3f5-11ea-918a-1234e695ba01.png)

  Fill details and click on create

  ![image](https://user-images.githubusercontent.com/68885738/90953480-80285200-e489-11ea-8ec1-0eedb4635efb.png)

  select security and give some name for token and then click on Generate
        3. Add SonarQube servers details with in "configure system"

![image](https://user-images.githubusercontent.com/68885738/90910714-689f8980-e3f5-11ea-889c-68e63b8302ce.png)

        Name: sonar-scanner
        Server URL: http://54.210.37.165:9000/
        Server authentication token: (Create secret text with authentication token)

**4. Generate kubeconfig to connect jenkins with EKs(for creating POD's)**

Configure Pod as slave:


jenkins/jnlp-slave
