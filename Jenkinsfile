pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Arshad-Sk/Ekart.git'
            }
        }

        stage('Code Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=EKART'''
                }
            }
        }

        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Push to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('Docker Build and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '6f40b747-c098-4ecb-9f77-5e71c9a78680', toolName: 'docker') {
                        sh "docker build -t sk77arshad/ekart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }

        stage('TRIVY') {
            steps {
                sh "trivy image sk77arshad/ekart:latest > trivy-report.txt"
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '6f40b747-c098-4ecb-9f77-5e71c9a78680', toolName: 'docker') {
                        sh "docker push sk77arshad/ekart:latest"
                    }
                }
            }
        }

        stage('Deploy to K8') {
            steps {
                withKubeConfig(
                    caCertificate: '',
                    clusterName: '',
                    contextName: '',
                    credentialsId: 'k8-token',
                    namespace: 'webapps',
                    restrictKubeConfigAccess: false,
                    serverUrl: 'https://172.31.2.190:6443'
                ) {
                    sh "kubectl apply -f deploymentservice.yml -n webapps"
                    sleep 30
                    sh "kubectl get svc -n webapps"
                }
            }
        }

    } // end of stages
} // end of pipeline
