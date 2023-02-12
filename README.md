# A simple Mini DevOps Project: 

In this mini project, we will create a jenkins pipeline by getting a SpringBoot Application code directly from Github and then create a Docker Image of it with the help of Jenkins. After Jenkins will push this Image to docker Hub and test our App by launching a Dcoker container of this Application.


![cover](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/cover.png)



### Prerequisites:
All we need to accomplish this project are:

- A springBoot Application availbale at: https://github.com/Tcarters/SpringBootApp_and_DevOps
- A Jenkins Server (**Mine is running in a local virtual machine at 192.168.38.90:8082** )
- A Docker service installed locally and running and a DockerHub Account (Mine is at https://hub.docker.com/search?q=tcdocker2021 ) . 

## Step 1: Launch The Jenkins server and Configure the Pipeline
 - Start by launching the Jenkins server 
![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-0.png)
 
 - Install the **Maven Integration** plugin because we will need it for pipeline Integration in upcoming steps.

![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/S1-00plugin.png)

 -  Create a new Item or Job as :
    - 1.1 Give a name to the new Job, here I use: ``springBt-docker-jenkins``
    - 1.2 Then, select the type as a  `` pipeline ``  
    - 1.3 After click **Ok** to save our choice.

![s1-1pic](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-1newjob.png)

- Configuration of the new Job:
    * On the `General ` section, give a description and a GitHub repo like showing below:
        ![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-2gene.png)

    * On the next section `Build Triggers`, select option "GitHub hook trigger for GITScm polling"
    ![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-3buildT.png)

    * Following the next section , we configure our pipeline stages. Feel free to use a jenkinsfile by pulling it from your gitHub repo or directly writing the script.
        - Before That configure your maven installation in Jenkins settings :    
          > In my case maven 3.6.3 is instatlled, and it can be accessed at: *Dashboard > Manage Jenkins > Global Tool Configuration*
        
            ![s1-4mvnConf](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-4mvnConf.png)

        - The pipeline script defined is :
        ![s1-5pipConf](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-5pipConf.png)

    * And after the pipeline Configured, we save our choice and go on Dashboard to build it.

    ![s1-6buildPip](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s1-6buildPip.png)

    - We can see that our build successed which means we have done with the first part of the project which is the Integration of Jenkins with our gitHub repoand setting up a pipeline.
  
## Step2: Tell jenkins to build the Docker Image for our Spring App
> Basically to build the docker Image, we all know that we need a Docker Image isnt ? 🤡 Okay so we have now to create a ``Dockerfile `` in our App repo and with help of Jenkins we'll generate a DockerImage ..Let's do it now 🚴

- 2.1  First create a file named `Dockerfile` in the main directory of src Spring App >> Refer to this link, to understand https://github.com/Tcarters/SpringBootApp_and_DevOps

- 2.2  In the same folder run  `mvn install` to update the `pom.xml` file of the Spring application. After commit and push it to your git Repo.
![s2-1push](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s2-0push.png)

- 2.3 Now before proceeding with jenkins, let's start the Docker service if it's not running . 
    > `As i am on linux machine, run : systemctl start docker`
    ![s1](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s2-1docker.png)

- 2.4  Now we have to modify our pipeline configuration to integrate another stage which will create a docker image. 
    - The new stage added in the pipeline script :
    ```
           stage ('Build SpringApp Docker Image ') {
            steps{
                script {
                    echo 'Checking if docker service is available ...'
                    sh 'systemctl is-active --quiet docker && echo "Service is running ..."'
                    echo 'Starting Docker Image building'
                    sh 'docker build -t tcdocker2021/springbt-in-docker:latest .'
                }
            }
        }
    ```
    - 📛📛 Important Configuration before executing the pipeline isrequired : we have to give permission to Jenkins to run docker service , e.g Error is out cause of that 
    ![s2-1builderror](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s2-2failbuild.png)

    - And after Restart the Jenkins service with ```sudo systemctl restart jenkins ```   
    - Now build the pipeline, after permission updated to Jenkins:
    
    ![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s2-3respip.png)
    ----
    ![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s2-4respip2.png)

## Step 3: Pushing the Docker Image with Jenkins help
> Now that Jenkins build the Docker Image for our application, we'll tell him to push the Image to our public docker Hub registry... And to accomplish it we have just to modify again our pipeline script and That's it.

- 3.1 Modifying the pipeline script by adding a new stage
    - The stage added is:
    ```
    stage ('Pushing the SpringApp Docker Image to Docker Registry') {
            steps {
                script {
                    echo 'Logging to Docker registry.....'
                    withCredentials([string(credentialsId: 'mytcdocker-hub', variable: 'mydockerhubpwd')]) {
                        sh 'docker login -u tcdocker2021 -p ${mydockerhubpwd}'    // some block
                    }
                    echo 'Starting the push of Docker Image ....'
                    sh ' docker push tcdocker2021/springbt-in-docker:latest '
                }
            }
        }
    ```
    - Before the push, we can see that we haven't a springApp pushed in our dockerhub registry.
    ![s3-0hub](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s3-0hub.png)

- 3.2 Save our changes and ` Build` again the pipeline we got :

    - On our Pipeline Dashboard View:  
     ![s3-1hub](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s3-1reshub.png)

    - By checking the log of our pipeline executed, we can see that everything go well.. 😀
    ![s3-2log](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s3-3coonsol.png)
    ![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s3-3consol2.png)

- 3.3 Checking our Docker Hub for verification : 
   ![s3-4hub](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s3-4hubproof.png)
   
   > So till now we can see our job is almost done but why not tell Jenkins to test our application ? So that will be our next travel ....

## Step 4: Testing our App by Launching a Docker container with Jenkins

- Here we are 🙂, To test our Application we will start a container based on the Image created earlier in a new state of our pipeline.

- The new stage is :
```
 stage ('Testing the deployment') {
            steps {
                script {
                    echo 'Starting a local container of the App ....'
                    sh 'docker run -dit --name springapp -p 2000:8080 tcdocker2021/springbt-in-docker:latest '
                    echo 'The App is now available at Port 2000 ....'
                }
            }
        }
```
- After saving and building again we have a new stage with a container launched for testing .
    ![s4-dashres](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s4-Dashpipres.png)

- Global View
![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s4-Dashpipres.png)

- On a local Browser, we can see our SpringApp available ...

![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/s4-dockRes.png)


## BONUS 
#### Creating a Jenkinsfile for our pipeline instead of editing the script many times ..

- To do it we have to create a file `Jenkinsfile` and put our pipeline code inside it.
- Content of `Jenkinsfile` :
```
pipeline {
    agent any
    tools {
        maven 'maven_3.6.3'
    }
    stages {
        stage('Build a Maven Project '){
            steps{ 
                echo 'Start The checking of github repository'
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Tcarters/SpringBootApp_and_DevOps']])
                echo 'Cleaning the Project'
                sh 'mvn clean install'
            }
        }
        stage ('Build SpringApp Docker Image ') {
            steps{
                script {
                    echo 'Checking if docker service is available ...'
                    sh 'systemctl is-active --quiet docker && echo "Service is running ..."'
                    echo 'Starting Docker Image building'
                    sh 'docker build -t tcdocker2021/springbt-in-docker:latest .'
                }
            }
        }
        stage ('Pushing the SpringApp Docker Image to Docker Registry') {
            steps {
                script {
                    echo 'Logging to Docker registry.....'
                    withCredentials([string(credentialsId: 'mytcdocker-hub', variable: 'mydockerhubpwd')]) {
                        sh 'docker login -u tcdocker2021 -p ${mydockerhubpwd}'    // some block
                    }
                    echo 'Starting the push of Docker Image ....'
                    sh ' docker push tcdocker2021/springbt-in-docker:latest '
                }
            }
        }
        stage ('Testing the deployment') {
            steps {
                script {
                    echo 'Starting a local container of the App ....'
                    sh 'docker run -dit --name springapp -p 2000:8080 tcdocker2021/springbt-in-docker:latest '
                    echo 'The App is now available at Port 2000 ....'
                }
            }
        }
    }
}
```
- And that summarize our pipeline code definition ...

- To test it, let's create another pipeline named `TestJenkinsfile` like and choose the pipeline method as *Git* with configuration like:

    - ![sbonus-1](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/sbonus-1.png)

    - ![sbonus-2](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/sbonus-2.png)

- After saved and build the configuration, we can see the success of App pushed, deployed and launched ...

    - ![sbonus-res](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/sbonus-res1.png)
    - ![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/sbonus-2.png)

- Verification on our browser, we have the App launched by jenkins

![sbonus-dockerOutput](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/sbonus-dockerOutput.png)

    - On Jenkins pipeline View Dashboard , we have:

![](https://github.com/Tcarters/mini-DevOps-Project_jenkins-springBoot-Docker/blob/master/Screenshots/sbonus-resDash.png)