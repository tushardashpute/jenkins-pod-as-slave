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

In the cluster, create a Namespace and ServiceAccount which will be used by Jenkins for authorization.

        $ kubectl apply -f jenkins_sa.yaml 
        namespace/test created
        serviceaccount/jenkins-slaves-service-account created
        rolebinding.rbac.authorization.k8s.io/jenkins-slaves-rolebinding created

**aws eks describe-cluster --name my-cluster --region us-east-2**

       # aws eks describe-cluster --name my-cluster --region us-east-2
        {
            "cluster": {
                "status": "ACTIVE", 
                "endpoint": "https://2D2B068677874DC557DC4D7A5A585436.gr7.us-east-2.eks.amazonaws.com", 
                "logging": {
                    "clusterLogging": [
                        {
                            "enabled": false, 
                            "types": [
                                "api", 
                                "audit", 
                                "authenticator", 
                                "controllerManager", 
                                "scheduler"
                            ]
                        }
                    ]
                }, 
                "name": "my-cluster", 
                "tags": {
                    "Name": "eksctl-my-cluster-cluster/ControlPlane", 
                    "alpha.eksctl.io/cluster-name": "my-cluster", 
                    "aws:cloudformation:stack-id": "arn:aws:cloudformation:us-east-2:462483370869:stack/eksctl-my-cluster-cluster/4a41e500-c5f3-11ec-a54e-06694700246a", 
                    "aws:cloudformation:logical-id": "ControlPlane", 
                    "aws:cloudformation:stack-name": "eksctl-my-cluster-cluster", 
                    "alpha.eksctl.io/eksctl-version": "0.94.0", 
                    "eksctl.cluster.k8s.io/v1alpha1/cluster-name": "my-cluster"
                }, 
                "certificateAuthority": {
                    "data": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1EUXlOekEyTXpReE1sb1hEVE15TURReU5EQTJNelF4TWxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBSnFyCk5rZnZXa1Y2a2R2cllsMlljWjNSbzh5WnVjNTJLaGIrelFrNnVZN2Vza29UbVk5SUtVWkp0aGFyaC9WRGRCeHEKakRZOGx3amdXb1FLMlc5NXBNWEd5b3M5VDNhY1JxSzNaUy85QW1CZFdxa2ZSdEkxTnJjUFJybXBjRjJ0N24zOApLQjFqc2wwRElaMDV5ZXdPYkJJVDlmSDIyTnJoZXI5VEN2bERPYU04QzQ2Qi9KcHJBYzRIVlZNOUYybEFtSnFrCjgyNHVreXZXRTZqN1IxTk1sU0dSaW9ha1d3QTZtcFVrZ3FmbUNPZFM4bmpQTlNVTktXVndIR2NhcGI4aUcvZEIKcVJSTThQcyt3azdWWlNFM2dsenBHMHREWG4rZjBscko2Vi8reTNid1RvRHRmSkQyZDhQcHJlcm5rVTc1NFZOQwpuRHV0VEJ3Z1VzTFNqWUhLWVZzQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZOSmM2TUxCMWo3STQ1cDNhVWkrVm15eHNkamNNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCei9IWnN0WW9wV0dRaE5WWHEwMDJONXFWNk5heUVPSXErODJZN1ZiSHBXZFdvS0JLMwpnUGVjUVJmU2RVOW93TThqQ2pIRzI1UTdLZis5OEdSeFMxeU53cTRtQ0lXazZVMy9VR2IwMG9WQkhlVWhtdS9tCmJFSFR4OWVuUWdoOHFWaThnRDh6WjM5dS9DTnZwWDUrWWhSaEZDRDdoeUFXaVllYUswMXl6amYrYUxMb1JTMTMKSUYrTGtibFdGRll2cForQUMvM1d6ZlFJaWVtR1lqbFZDTnZGejIrTlpwMFV1MHpWcXFQSXNibzdIYkdTVWtMNgoxZkszdXBhY0Fqc0VwSTUyNkY5MnhFTVhkdWozOXdIOHUvZ1dTc0tpVjJwNUpyTW1DTUxET0J6bmpzOFhZVnJHCnc4dEtiZE5FWkFBdVRmQ2daNG8xdU5oRWRWbXh0V0tKd0xNOAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
                }, 
                "roleArn": "arn:aws:iam::462483370869:role/eksctl-my-cluster-cluster-ServiceRole-1RGFNJ3RD10ZY", 
                "kubernetesNetworkConfig": {
                    "serviceIpv4Cidr": "10.100.0.0/16"
                }, 
                "resourcesVpcConfig": {
                    "vpcId": "vpc-07d99b3190943ed33", 
                    "subnetIds": [
                        "subnet-0f130425ce56e4d37", 
                        "subnet-07c2d6bc0827aa8b6", 
                        "subnet-0ed331ff01092a69a", 
                        "subnet-05be685ee4e950bf3", 
                        "subnet-0f226ac5e7ae89efd", 
                        "subnet-0fad5669b7fe59df2"
                    ], 
                    "securityGroupIds": [
                        "sg-06442399bfa1a0b35"
                    ], 
                    "clusterSecurityGroupId": "sg-0d8214c0dce70e1d0", 
                    "publicAccessCidrs": [
                        "0.0.0.0/0"
                    ], 
                    "endpointPublicAccess": true, 
                    "endpointPrivateAccess": false
                }, 
                "platformVersion": "eks.6", 
                "version": "1.21", 
                "arn": "arn:aws:eks:us-east-2:462483370869:cluster/my-cluster", 
                "identity": {
                    "oidc": {
                        "issuer": "https://oidc.eks.us-east-2.amazonaws.com/id/2D2B068677874DC557DC4D7A5A585436"
                    }
                }, 
                "createdAt": 1651040959.184
            }
        }
       
 Now get the secrets and decode it for the SA which we have created.
 
 **kubectl get secrets -n test**
 
        [root@ip-172-31-13-254 opt]# kubectl get secrets -n test
        NAME                                         TYPE                                  DATA   AGE
        default-token-757qn                          kubernetes.io/service-account-token   3      2m36s
        jenkins-slaves-service-account-token-z5dpt   kubernetes.io/service-account-token   3      2m36s
 
 Decode the secret token which we got.
 
 **k get secrets -n test jenkins-slaves-service-account-token-z5dpt -o jsonpath='{.data.token}' | base64 --decode**
        
        [root@ip-172-31-13-254 opt]# kubectl get secrets -n test jenkins-slaves-service-account-token-z5dpt -o jsonpath='{.data.token}' | base64 --decode
        eyJhbGciOiJSUzI1NiIsImtpZCI6IkV5MkdFbEtQZTMzcWw3ZGtVdjdTZmpicTBXZG1nR1k5SlMyc3JPaTlXMzgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ0ZXN0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImplbmtpbnMtc2xhdmVzLXNlcnZpY2UtYWNjb3VudC10b2tlbi16NWRwdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zLXNsYXZlcy1zZXJ2aWNlLWFjY291bnQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2ZjJlNzdiMC0yOGZlLTQ1YzYtYjRiZS1mMjU4ZDcxZWI2NDciLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6dGVzdDpqZW5raW5zLXNsYXZlcy1zZXJ2aWNlLWFjY291bnQifQ.VvK-GAc3AaSMEa-iI10reH89s7PSzpay--AwTRc32Du8wU210Tfg19EiwT14eaVd-Oz3YXHC69cfbZARs_DDA0IaquEN3bGvN_a-VSTqzQtfMcpjvjZM0DimtWVVs5ohJHv1pQA4TSuzN5CfMuT5NZCLRI-ZCI1KFwjovQf4P_qtEme7h8kNsZZ9S31qPj4VGYJaiIWHpwkgMH_yFO__w_4DxvSoOw2WLlJS6bGdUd47RvD09-u_U8U0E0slo_QPaMsnhoRZaAns86npn4dm0GXMlm555n8ARK9bnFCJQ33ekZAl1rzUQAms-dGClk3IbGDuSb4CsZ6u39Ptur7HRA

Configure Pod as slave:


jenkins/jnlp-slave
