pipeline {
    agent any

    tools {
        maven "maven3"
        jdk "jdk-17"
    }
    
    environment {
        RELEASE = "1.0.0"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        DOCKER_REGISTRY = "ghkimdev"
        DOCKER_REPOSITORY = "spring-petclinic"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }

    stages {
        stage("Notify Start to Slack") {
            steps {
                script {
                    slackSend channel: 'jenkins-noti', color: 'good', message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} Started by ${env.USER} (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack-credential'
                }
            }
        }
        stage('Checkout Source Code') {
            steps {
                git branch: 'main', credentialsId: 'git-credential', url: 'https://github.com/ghkimdev/spring-petclinic-CICD-Pipeline.git'
            }
        }
        stage('Build & Unit Tests') {
            steps {
                script {
                    withMaven(maven:'maven3') {
                        sh './mvnw -B verify'
                    }
                }
            }
        }
        stage('Static Code Analysis with SonarQube') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        withMaven(maven:'maven3') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }
            }
        }
        stage("Quality Gate Check") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }
        }
        stage("Dependency Scanning with OWASP Dependency-check") {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dp-check', stopBuild: false
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                    sh 'rm -f dependency-check-report.xml' 
                }
            }
        }
        stage("File System Vulnerability Scan with Trivy") {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --no-progress --format table -o trivy-fs-report.html .'
                sh 'rm -f trivy-fs-report.html'
            }
        }
        stage("Nexus Artifact Upload") {
            steps {
                script {
                    nexusPublisher nexusInstanceId: 'nexus', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: '/var/lib/jenkins/workspace/spring-petclinic-ci/target/spring-petclinic-3.4.0-SNAPSHOT.jar']], mavenCoordinate: [artifactId: 'com.demo.spring-petclinic', groupId: 'petclinic', packaging: 'jar', version: '1.0.0']]]
                    sh 'rm -rf target/'
                }
            }
        }
        stage("Docker Build") {
            steps {
                sh './mvnw spring-boot:build-image'
            }
        }
        stage("Docker Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-credential') {
                        sh 'docker tag ${DOCKER_REPOSITORY}:3.4.0-SNAPSHOT ${DOCKER_REGISTRY}/${DOCKER_REPOSITORY}:${IMAGE_TAG}'
                        sh 'docker push "${DOCKER_REGISTRY}/${DOCKER_REPOSITORY}:${IMAGE_TAG}"'
                        sh 'docker rmi "${DOCKER_REPOSITORY}:3.4.0-SNAPSHOT"'
                        sh 'docker rmi "${DOCKER_REGISTRY}/${DOCKER_REPOSITORY}:${IMAGE_TAG}"'
                    }
                }
            }
        }
        stage("Container Image Vulnerability Scan with Trivy") {
            steps {
                sh 'trivy image ${DOCKER_REGISTRY}/${DOCKER_REPOSITORY}:${IMAGE_TAG} --format table -o trivy-image-report.html'
                sh 'rm -f trivy-image-report.html'
            }
        }
        stage("Kubernetes Deployment Update") {
            steps {
                sh """
                   cat k8s/petclinic.yml
                   sed -i 's/${DOCKER_REPOSITORY}:.*/${DOCKER_REPOSITORY}:${IMAGE_TAG}/g' k8s/petclinic.yml
                   cat k8s/petclinic.yml
                """
            }
        }

        stage("Push the changed deployment file to Git") {
            steps {
                sh """
                    git config --global user.name "ghkimdev"
                    git config --global user.email "ghkim.dev@gmail.com"
                    git add k8s/petclinic.yml
                    git commit -m "Update petclinic.yml"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'git-credential', gitToolName: 'Default')]) {
                    sh 'git push https://github.com/ghkimdev/spring-petclinic-CICD-Pipeline.git main'
                }
            }
        } 
        
    }
    post {
        success {
            slackSend channel: 'jenkins-noti', color: 'good', message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} ${currentBuild.result} after ${currentBuild.durationString} sec (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack-credential'
        }
        failure {
            slackSend channel: 'jenkins-noti', color: 'danger', message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} ${currentBuild.result} after ${currentBuild.durationString} sec (<${env.BUILD_URL}|Open>)", tokenCredentialId: 'slack-credential'
        }
    }
}

