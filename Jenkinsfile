pipeline {
    agent any 
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-token'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Code Checkout From Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Cloud-Gen-DevOps-Projects/AWS-Terraform-Zomato-Project.git'
            }
        }
        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar-token') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage('Code Quality Gates') {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                    }
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage('Docker Image Build and Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t coolfaiz/cloudzomato ."
                        sh "docker tag coolfaiz/cloudzomato:latest coolfaiz/cloudzomato:latest"
                        sh "docker push coolfaiz/cloudzomato:latest"
                    }
                }
            }
        }
        stage('TRIVY Image Scanning') {
            steps {
                sh "trivy image coolfaiz/cloudzomato:latest > trivy.txt"
            }
        }
        stage('Creating Docker Container') {
            steps {
                sh 'docker run -d --name zomato-app -h zomato -p 3000:3000 coolfaiz/cloudzomato:latest'
            }
        }
    }
}

