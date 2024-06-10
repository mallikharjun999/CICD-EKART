pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
}
    stages{
        stage ('GIT CHECKOUT'){
            steps{
                git branch: 'main' , url: 'https://github.com/mallikharjun999/CICD-EKART.git'
            }
        }

        stage ('COMPILE'){
            steps{
                sh "mvn compile"
            }
        }

         stage ('Trivy FS Scan'){
            steps{
                sh "trivy fs ."
            }
        }

        stage ('OWASP FILE SYSTEM SCAN'){
            steps{
                dependencyCheck additionalArguments: '--scan ./' , odcInstallation:'DC'
                    dependencyCheckPublisher pettern: '**/dependency-check-report.xml'
            }
        }

        stage ('SONARQUBE ANALISYS'){
            steps{
                withSonarQubeEnv('sonar'){

                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=EKART '''
            }
        }
        }

        stage ('BUILD'){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }

        stage ('Deploy to nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'global-settings-xml'){
                sh "mvn deploy -DskipTests=true"
                 }
            }
        }
        stage ('BUILD & TAG DOCKER IMAGE '){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred' , toolName: 'docker'){
                        sh "docker build -t mycartapp:latest -f docker/Dockerfile ."
                        sh "docker tag mycartapp:latest mallikharjun999/mycart"
                    }
                }
            }
        }

        stage ('PUSH DOCKER IMAGE '){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred' , toolName: 'docker'){
                        sh "docker push mallikharjun999/mycart "
                        
                    }
                }
            }
        }

        stage ('DEPLOY APPLICATION '){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred' , toolName: 'docker'){
                        sh "docker run -d --name ekart -p 8070:8070 mallikharjun999/mycart"
                        
                    }
                }
            }
        }
    }