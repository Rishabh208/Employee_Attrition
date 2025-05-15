pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS' // Configure in Jenkins Global Tool Configuration
        jdk 'JDK'       // Configure in Jenkins Global Tool Configuration
    }
    
    environment {
        DOCKER_HUB_CREDS = credentials('DockerHubCred')
        DOCKER_IMAGE_BACKEND = 'siddhesh01/ml-backend'  // Change 'yourusername'
        DOCKER_IMAGE_FRONTEND = 'siddhesh01/ml-frontend'  // Change 'yourusername'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Backend Tests') {
            steps {
                dir('backend') {
                    sh 'python3 -m pip install --upgrade pip'
                    sh 'pip3 install -r requirements.txt'
                    sh 'python3 manage.py test'
                }
            }
        }
        
        stage('Frontend Tests') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm test -- --passWithNoTests'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo 'SonarQube analysis temporarily disabled'
                // Commenting out the SonarQube integration until the plugin is installed
                // withSonarQubeEnv('SonarQubeServer') {
                //     sh 'sonar-scanner'
                // }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG} ./backend'
                sh 'docker build -t ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG} ./frontend'
                sh 'docker tag ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG} ${DOCKER_IMAGE_BACKEND}:latest'
                sh 'docker tag ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG} ${DOCKER_IMAGE_FRONTEND}:latest'
            }
        }
        
        stage('Push Docker Images') {
            steps {
                sh 'echo ${DOCKER_HUB_CREDS_PSW} | docker login -u ${DOCKER_HUB_CREDS_USR} --password-stdin'
                sh 'docker push ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG}'
                sh 'docker push ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG}'
                sh 'docker push ${DOCKER_IMAGE_BACKEND}:latest'
                sh 'docker push ${DOCKER_IMAGE_FRONTEND}:latest'
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                sh '''
                sed -i "s|image: ${DOCKER_IMAGE_BACKEND}:.*|image: ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG}|g" kubernetes/backend-deployment.yaml
                sed -i "s|image: ${DOCKER_IMAGE_FRONTEND}:.*|image: ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG}|g" kubernetes/frontend-deployment.yaml
                '''
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig', namespace: 'ml-app']) {
                    sh 'kubectl apply -f kubernetes/'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig', namespace: 'ml-app']) {
                    sh 'kubectl rollout status deployment/backend -n ml-app'
                    sh 'kubectl rollout status deployment/frontend -n ml-app'
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
            cleanWs()
        }
        success {
            echo 'Pipeline executed successfully!'
            // Remove or comment out the slackSend line
        }
        failure {
            echo 'Pipeline execution failed!'
            // Remove or comment out the slackSend line
        }
    }
}