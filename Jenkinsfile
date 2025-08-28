pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'Maven3'
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/Apratim1991/Boardgame.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
         stage('Test') {
            steps {
                sh "mvn test"
            }
         }
          stage('Security Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
         }
         stage('SonarScanner Analysis') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                sh '''${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                -Dsonar.java.binaries=.'''
                }
            }
         }
          stage('Build') {
            steps {
                sh "mvn package"
            }
         }
           stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'Maven3', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy"
                }
            }
         }
          stage('Docker Build') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred',toolName: 'docker') {
                     sh "docker build -t apratim1991/boardgame:latest ."
                        }
                    }
                }
            }
          stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html apratim1991/boardgame:latest"
            }
         }
          stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred',toolName: 'docker') {
                     sh "docker push apratim1991/boardgame:latest"
                        }
                    }
                }
            }
             stage('Deploy to K8 Cluster') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.0.10:6443') {
                    sh "kubectl apply -f deploymentservice.yaml"
                     }
                }
            }
             stage('Validate Deployment') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.0.10:6443') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                     }
                }
            }
         }
    }
