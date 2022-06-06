pipeline {
    agent any

    tools {
        maven "MAVEN"
    }
    environment{
        COMMIT = ""
        DATE = new Date().format('M-yy')
    }

    stages {
        stage('SonarQube Analysis') {
            steps{
                script{
                    withSonarQubeEnv(installationName: "sonarqube") {
                        bat "mvn clean test verify package sonar:sonar -Dsonar.projectKey=underwriter-microservice-kdl"
                        }
                }

            }       
        }

        stage('Quality Gate Check'){
            steps{
                script{
                    sleep(5)
                        def qg = waitForQualityGate()
                        if(qg.status != 'OK'){
                            error "Pipeline aborted due to quality gate failure"
                    }
                    }
                }
                
            }
        
        stage("Build w/MVN") {
            steps {
                bat "mvn clean package -Dskiptests"
            }
        }

        stage("Build Image w/docker"){
            steps{
                script{
                    COMMIT = "${GIT_COMMIT}"
                    SLICE = COMMIT[1..7]
                    bat "docker build -t $env.AWS_ECR_REGISTRY/underwriter-microservice-kdl:${SLICE}.${BUILD_NUMBER}.${DATE} ."
                }


            }            
        }

        stage("Deploy to AWS"){
            steps{
                script{
                    COMMIT = "${GIT_COMMIT}"
                    SLICE = COMMIT[1..7]
                   docker.withRegistry("https://$env.AWS_ECR_REGISTRY", "ecr:$env.AWS_REGION:AWS"){
                       docker.image("$env.AWS_ECR_REGISTRY/underwriter-microservice-kdl:${SLICE}.${BUILD_NUMBER}.${DATE}").push()
                   }
                }
            }
        }
            
        stage("Cleaning"){
            steps{
                bat "docker system prune --all -f"
            }
        }
        
    }
    }
