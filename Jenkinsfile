pipeline {
    agent any
    
    tools {
        maven 'maven3.6'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME = tool 'sonarqube - scipple'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'Checkout code'
                git branch: 'main', url: 'https://github.com/jagrit007/eKart-DEVOPS-CICD'
            }
        }
        stage('Compile Source Code') {
            steps {
                echo 'Compiling the source code...'
                sh "mvn compile"
            }
        }
        stage('Unit Tests') {
            steps {
                echo 'We are going to skip test cases for this project!!'
                sh "mvn test -DskipTests=true"
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=eKart -Dsonar.projectName=Ekart -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'dc-6.5'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3.6', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('Build and Tag Container Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'gitea-cr', url: 'https://gitea.jagrit.dev') {
                        sh "docker build -t gitea.jagrit.dev/jagrit007/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh "trivy image gitea.jagrit.dev/jagrit007/ekart:latest > trivy-report.txt"
            }
        }
        stage('Push docker image to container registry') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'gitea-cr', url: 'https://gitea.jagrit.dev') {
                        sh "docker push gitea.jagrit.dev/jagrit007/ekart:latest"
                    }
                }
            }
        }
    }
}
