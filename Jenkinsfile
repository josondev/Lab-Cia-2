pipeline {
    agent any

    stages {
        stage('Build and Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn clean verify'
                    } else {
                        bat 'mvn clean verify'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker build -t campus-equipment-booking:set-d .'
                    } else {
                        bat 'docker build -t campus-equipment-booking:set-d .'
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            minikube start
                            minikube image load campus-equipment-booking:set-d
                            kubectl apply -f deployment.yaml
                            kubectl rollout status deployment/equipment-booking-deployment --timeout=180s
                            kubectl scale deployment equipment-booking-deployment --replicas=2
                            kubectl rollout status deployment/equipment-booking-deployment --timeout=180s
                            kubectl get pods -l app=equipment-booking -o wide
                        '''
                    } else {
                        bat '''
                            minikube start
                            minikube image load campus-equipment-booking:set-d
                            kubectl apply -f deployment.yaml
                            kubectl rollout status deployment/equipment-booking-deployment --timeout=180s
                            kubectl scale deployment equipment-booking-deployment --replicas=2
                            kubectl rollout status deployment/equipment-booking-deployment --timeout=180s
                            kubectl get pods -l app=equipment-booking -o wide
                        '''
                    }
                }
            }
        }
    }
}