pipeline{
    agent any
    tools {
        jdk "Java11"
        maven "mvn"
    }

    stages{
        stage("fetch form git"){
            steps{
                git branch: "SCA-VP", url: "https://github.com/divzraj/devprojects.git"
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
}
}