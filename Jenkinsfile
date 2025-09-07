pipeline {
    agent any
    
<<<<<<< HEAD
    tools {
        nodejs 'NodeJS'
        jdk 'JDK'     
    }
    
    environment {
        DOCKER_IMAGE_BACKEND = 'rksingh5/ml-backend'  
        DOCKER_IMAGE_FRONTEND = 'rksingh5/ml-frontend'  
=======
    environment {
        GITHUB_REPO_URL = 'https://github.com/Rishabh208/Employee_Attrition.gitqewcx'
        DOCKER_HUB_CREDS = credentials('DockerHubCred')
        DOCKER_IMAGE_BACKEND = 'rksingh5/ml-backend'
        DOCKER_IMAGE_FRONTEND = 'eksingh5/ml-frontend'
>>>>>>> fae1309 (fix)
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG_CRED = credentials('mykubeconfig')
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                script {
                    // clone the code from the GitHub repository
                    git branch: 'main', url: "${GITHUB_REPO_URL}"
                }
            }
        }
        
<<<<<<< HEAD
        stage('SonarQube Analysis') {
            steps {
                echo 'SonarQube analysis temporarily disabled'
            }
        }
        
=======
>>>>>>> fae1309 (fix)
        stage('Build Docker Images') {
            steps {
                dir('Employee_Attrition/backend') {
                    sh 'docker build -t ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG} .'
                    sh 'docker tag ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG} ${DOCKER_IMAGE_BACKEND}:latest'
                }
                dir('Employee_Attrition/frontend') {
                    sh 'docker build -t ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG} .'
                    sh 'docker tag ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG} ${DOCKER_IMAGE_FRONTEND}:latest'
                }
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
                sed -i "s|image: ${DOCKER_IMAGE_BACKEND}:.*|image: ${DOCKER_IMAGE_BACKEND}:${DOCKER_TAG}|g" Employee_Attrition/k8s/backend-deployment.yaml
                sed -i "s|image: ${DOCKER_IMAGE_FRONTEND}:.*|image: ${DOCKER_IMAGE_FRONTEND}:${DOCKER_TAG}|g" Employee_Attrition/k8s/frontend-deployment.yaml
                sed -i 's/imagePullPolicy: Never/imagePullPolicy: Always/g' Employee_Attrition/k8s/backend-deployment.yaml
                sed -i 's/imagePullPolicy: Never/imagePullPolicy: Always/g' Employee_Attrition/k8s/frontend-deployment.yaml
                '''
            }
        }
        
        stage('Deploy with Ansible') {
            steps {
                withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    dir('Employee_Attrition') {
                        sh '''
                        export ANSIBLE_PYTHON_INTERPRETER=/usr/bin/python3
                        export KUBECONFIG="${KUBECONFIG_FILE}"
                        cat "${KUBECONFIG_FILE}" > /tmp/kubeconfig
                        chmod 600 /tmp/kubeconfig
                        ansible-playbook -i ansible/inventory -e "kubeconfig=/tmp/kubeconfig" ansible/deploy.yml
                        '''
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh '''
                    KUBECONFIG=${KUBECONFIG_FILE} kubectl get pods -n employee-attrition
                    KUBECONFIG=${KUBECONFIG_FILE} kubectl get services -n employee-attrition
                    KUBECONFIG=${KUBECONFIG_FILE} kubectl get hpa -n employee-attrition
                    '''
                }
            }
        }
    }
    
    post {
        always {
            sh 'rm -f /tmp/kubeconfig || true'
            cleanWs()
        }
        success {
<<<<<<< HEAD
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline execution failed!'
=======
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
>>>>>>> fae1309 (fix)
        }
    }
}