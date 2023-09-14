pipeline {
    agent any
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git 'https://github.com/EmAdd9/.Net-DemoApp.git'
            }
        }
        stage('Dependency check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=dotnet-demoapp \
                    -Dsonar.projectKey=dotnet-demoapp '''
                }

            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '4f432487-96dc-4ffc-acd7-c24971224f65', toolName: 'docker') {
                        sh "make image"
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image sudebdocker/dotnet-demoapp:latest"
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '4f432487-96dc-4ffc-acd7-c24971224f65', toolName: 'docker') {
                        sh "make push"
                    }
                }
            }
        }
        stage('Docker Container Deploy') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '4f432487-96dc-4ffc-acd7-c24971224f65', toolName: 'docker') {
                        sh "docker run -d --name dotnet-cont -p 5000:5000 sudebdocker/dotnet-demoapp:latest"
                    }
                }
            }
        }
    }
}
