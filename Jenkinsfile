pipeline {
    agent any

    environment {
        JAVA_HOME = "/usr/lib/jvm/java-21-amazon-corretto.x86_64"
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        AWS_REGION = 'us-east-1'
        ECR_REPO  = '881601365147.dkr.ecr.us-east-1.amazonaws.com/boardgame'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarScanner') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=boardgame -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: 'http://52.54.249.53:8081/',
                    repository: 'maven-jenkins',
                    credentialsId: 'nexus-creds',
                    groupId: 'com.example',
                    version: '1.0.${BUILD_NUMBER}',
                    artifacts: [[artifactId: 'boardgame', classifier: '', file: 'target/database_service_project-0.0.7-SNAPSHOT.jar', type: 'jar']]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t boardgame:${IMAGE_TAG} .'
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker tag boardgame:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        sed -i 's#IMAGE_TAG#${IMAGE_TAG}#g' deployment-service.yaml
                        kubectl apply -f deployment-service.yaml
                        kubectl rollout status deployment/boardgame
                    """
                }
            }
        }

    }
}
