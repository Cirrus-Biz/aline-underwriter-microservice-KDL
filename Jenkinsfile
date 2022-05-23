pipeline {
    environment {
        DOCKERHUB_CREDENTIALS=credentials("Dockerhub")
    }
    agent any

    tools {
        maven "MAVEN"
    }

    stages {
        stage("Build MVN") {
            steps {
                bat "mvn -Dmaven.test.failure.ignore=true clean install"
            }
        }

        stage('SonarQube Analysis') {
            steps{
                    withSonarQubeEnv(installationName: "sonarqube") {
                        bat "mvn sonar:sonar"
                        }

                    // sleep(5)
                    // def qg = waitForQualityGate()
                    // if (qg.status != "OK"){
                    //     error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    // }

            }       
        }


        stage("Build Docker"){
            steps{
                bat "docker build -t laxwalrus/capstone-underwriter:$BUILD_NUMBER ."

            }            
        }


        stage("login to docker"){
            steps{
                bat "echo $DOCKERHUB_CREDENTIALS_PSW| docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
            }
        }

        stage("Deploy Docker"){
            steps{
                bat "docker push laxwalrus/capstone-underwriter:$BUILD_NUMBER"
            }
        }
            
        stage("Cleaning"){
            steps{
                bat "docker logout"
            }
        }
  
        stage('Archive') {
            steps {
            archiveArtifacts artifacts: 'underwriter-microservice/target/*.jar', followSymlinks: false
            }
        }
    }
}