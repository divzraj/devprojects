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
    }
}