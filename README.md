# project-CI

💻This repository aims to store a CI pipeline code that automates the entire continuous integration and delivery process of a web application built in Java. It performs the following steps:
- Project construction
- Test execution
- Application deployment
- Code quality analysis via SonarQube
- Artifact storage in Nexus
- Completion notification via Slack

✏️ The diagram below better illustrates this flow:
[![Ci Diagram](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaCI.png "Ci Diagram")](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaCI.pnghttp:// "Ci Diagram")

☁️ This CI/CD pipeline is fully compatible with cloud construction, in this case we will be using the AWS cloud computing platform. It is important to visualize the project as a whole so that we can acquire the correct instances and services to avoid unnecessary costs.

[![AWS Diagram](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaAWS.png "AWS Diagram")](https://project-ci-repository.s3.sa-east-1.amazonaws.com/DiagramaAWS.png "AWS Diagram")

## Initialization and configuration of servers.
###Step 01 - Set up servers
To set up the three servers, simply navigate to the ** /Setups** directory and run the startup script for each one within AWS. It's important to note that the SonarQube and Jenkins servers will be booted on an Ubuntu Server 24 image, while the rest will be booted on an Amazon Linux 23.

###Step 02 - Configure security groups
In the flowchart we can see the entry rules for each security group. We need to pay attention to these rules, because if they are not configured correctly the servers will not be able to communicate with each other.

###Step 03 - SonarQube analysis
The first step is to create a step that will analyze our code

```groovy
    stage('Test') {
                steps {
                    sh 'mvn test'
                }
            }
	stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
```

When we run this code, Maven will perform a code analysis for us using the checkstyle plugin. It will download the dependencies, run it, and generate the result.
After that, the results will be unique in the /targets folder of our Jenkins environment. The file name will be "checkstyle-result.xml". However, the file will be in XML format, which is unreadable for humans, so we can use the SonarQube dashboard to publish and view the results. Therefore, we will create a step that will send to SonarQube.

```groovy
stage("Sonar Code Analysis") {
        	environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
              withSonarQubeEnv('sonarserver') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
```
**OBS: For academic purposes only, we will not make any changes to the Quality Gates.**
###Step 04 - Configure Nexus Repository

Nexus is a repository manager used to store, organize and distribute software artifacts, such as libraries, packages and containers. We will store it using a new step in our pipeline.

```groovy
stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: 'your.nexusserverprivate.ip:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'vprofile-repo',
                  credentialsId: 'nexuslogin',
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
```

OBS: You will create a maven2(hosted) type repository, and don't forget to configure the nexus credentials within your Jenkins, so that it can make changes within the server.

###Step 05 - Configure Slack Notification (optional)

To finalize our pipeline, we can set up a notification via Slack to see whether the pipeline ran successfully or not.

```groovy
post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#your-channel',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
```
We added an extra configuration at the beginning of the Jenkinsfile

```groovy
def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
```
After that, your pipeline will go through all the steps described.
