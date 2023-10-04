pipeline{
    agent any
    tools {
        jdk "java8"
        maven "mvn"
    }
    environment{
        regioncreds = "ecr:us-east-1:awscreds"
        Imagename = "587939292877.dkr.ecr.us-east-1.amazonaws.com/myprojimg"
        vproecrregistry = "https://587939292877.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages{
        stage("fetch form git"){
            steps{
                git branch: "docker", url: "https://github.com/divzraj/devprojects.git"
            }

        }

        stage('maven build'){
            steps{
                sh 'mvn clean install -DskipTests'
            }
            post{
                success{
                    echo " archiving"
                    archiveArtifacts artifacts: '**/*.war'

                }
            }

        }

        stage("Test"){
            steps{
                sh 'mvn test'
            }
        }

        stage("checkstyle anaylsis"){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonarqube analysis'){
            environment {
            scannerHome = tool 'sonar4.7'
           }
            steps{
                withSonarQubeEnv('sonarserver') {
                 sh ''' ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=proj1 \
                   -Dsonar.projectName=proj1 \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                   '''
                }
        }
    }

    stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
          }

    stage("upload to Nexus"){
        steps{
            nexusArtifactUploader(
            nexusVersion: 'nexus3',
            protocol: 'http',
            nexusUrl: '172.31.63.220:8081',
            groupId: 'test',
            version: "${env.BUILD_ID}_${env.BUILD_TIMESTAMP}",
            repository: 'projectstores',
            credentialsId: 'nexuslogin',
            artifacts: [
            [artifactId: 'projvpro',
             classifier: '',
             file: 'target/vproj-v2.war',
             type: 'war']
        ]
     )
        }
    }

    stage('Image build'){
        steps{
            script{
                Dockerimage = docker.build( "${Imagename}" + ":$BUILD_NUMBER" , "./Docker-files/app/" )
            }
        }
    }

    stage("upload image to registry in aws"){
        steps{
            script{
                docker.withRegistry( "${vproecrregistry}", "${regioncreds}" ) {
                        // Push the Docker image to ECR
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push("latest")
            }
        }
    }
}
}