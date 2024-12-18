pipeline {
    agent any

    tools {
        nodejs 'nodejs'  // Ensure nodejs is properly configured in Jenkins
        //maven 'maven'  // Ensure Maven is properly configured in Jenkins
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'  // Keep this if you plan to re-enable SonarQube in the future
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Use Git SCM to check out code from a public repository
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']], // Replace 'main' with your branch name
                    userRemoteConfigs: [[
                        url: 'https://github.com/Sriram8788/docker-frontend-backend-db.git' // Public repository URL
                    ]]
                ])
            }
        }

        /*stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
        }*/

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t web-image frontend/."
                        sh "docker build -t api-image backend/."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html sriram8788/camp:latest"
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push sriram8788/camp:latest"
                    }
                }
            }
        }

        stage('Docker Deploy To Local') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker network create network-backend"
                        sh "docker volume create mongodb_data"
                        sh "docker run -d --name mongo --restart always --network network-backend -v mongodb_data:/data/db -e MONGO_INITDB_ROOT_USERNAME=username -e MONGO_INITDB_ROOT_PASSWORD=password mongo"
                        sh "docker run -d --name api --network network-backend -p 3001:3001 -e MONGO_URI="mongodb://username:password@mongo:27017" api-image"
                        sh "docker run -d --name web --network network-backend -p 3000:3000 -e API_BASE_URL="http://localhost:3001" web-image"
                    }
                }
            }
        }

        /*stage('Deploy To EKS') {
            steps {
                withKubeCredentials(kubectlCredentials: [
                    [caCertificate: '', clusterName: 'my-eks22', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://80463D4E61B703ABB582BE4481F43614.yl4.us-east-1.eks.amazonaws.com']
                ]) {
                    sh "kubectl apply -f Manifests/"
                    sleep 60 // Wait for the deployment to stabilize
                }
            }
        }

        stage('Verify the Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [
                    [caCertificate: '', clusterName: 'my-eks22', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://80463D4E61B703ABB582BE4481F43614.yl4.us-east-1.eks.amazonaws.com']
                ]) {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }*/
    }
}

