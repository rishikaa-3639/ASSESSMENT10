pipeline {
    agent any
    parameters {
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run code verification tests')
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Parallel Quality Check') {
            parallel {
                stage('Validate HTML') {
                    steps {
                        echo "Validating HTML structure..."
                        sh 'test -f index.html'
                    }
                }
                stage('Check Dockerfile') {
                    steps {
                        echo "Checking Dockerfile configuration..."
                        sh 'test -f Dockerfile'
                    }
                }
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                script {
                  
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-cred') {
                        def customImage = docker.image("dockerhubusername/equipment-monitor:${env.BUILD_NUMBER}")
                        customImage.build()
                        customImage.push()
                        customImage.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    try {
                        sh 'kubectl apply -f deployment.yaml'
                        sh 'kubectl rollout status deployment/equipment-monitor-app'
                    } catch (Exception err) {
                        currentBuild.result = 'FAILURE'
                        echo "Deployment failed! Initiating automated rollback..."
                        sh 'kubectl rollout undo deployment/equipment-monitor-app'
                        error("Deployment failed, previous stable version restored.")
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
