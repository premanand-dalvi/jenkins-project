pipeline {
    agent any

    environment {
        DOCKER_USER  = "premananddalvi"
        BACKEND_IMG  = "${DOCKER_USER}/jenkins-proj-backend"
        FRONTEND_IMG = "${DOCKER_USER}/jenkins-proj-frontend"
    }

    stages {
        stage('1. Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/premanand-dalvi/jenkins-project.git'
            }
        }

        stage('2. SonarQube Code Scan') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    sonar-scanner \
                      -Dsonar.projectKey=jenkins-project \
                      -Dsonar.sources=backend,frontend
                    '''
                }
            }
        }

        stage('3. Build Docker Images') {
            parallel {
                stage('Backend Image') {
                    steps {
                        dir('backend') {
                            sh "docker build -t ${BACKEND_IMG}:${BUILD_NUMBER} ."
                        }
                    }
                }
                stage('Frontend Image') {
                    steps {
                        dir('frontend') {
                            sh "docker build -t ${FRONTEND_IMG}:${BUILD_NUMBER} ."
                        }
                    }
                }
            }
        }

        stage('4. Trivy Security Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL ${BACKEND_IMG}:${BUILD_NUMBER}"
                sh "trivy image --severity HIGH,CRITICAL ${FRONTEND_IMG}:${BUILD_NUMBER}"
            }
        }

        stage('5. Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push ${BACKEND_IMG}:${BUILD_NUMBER}"
                        sh "docker push ${FRONTEND_IMG}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('6. Deploy Live Website') {
            steps {
                sh '''
                docker rm -f live-backend live-frontend || true
                docker network create prod-net || true
                docker run -d --name live-backend --network prod-net -p 5000:5000 ${BACKEND_IMG}:${BUILD_NUMBER}
                docker run -d --name live-frontend --network prod-net -p 3000:80 ${FRONTEND_IMG}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        always {
            sh 'docker image prune -f'
        }
        success {
            echo 'cong http://localhost:3000 website are run finally'
        }
    }
}
