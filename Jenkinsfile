pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ghcr.io/rayhanardiansyah17/cicdtest.git:latest'
        DOCKER_CREDENTIALS_ID = 'PAT_CERT'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/rayhanardiansyah17/testcicd.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -f Dockerfile -t ghcr.io/rayhanardiansyah17/cicdtest:latest .'
                }
            }
        }
        
        stage('Test Nginx App') {
            steps {
                script {
                    // Stop and remove the container if it exists
                    sh '''
                        if [ $(docker ps -aq -f name=nginx-website) ]; then
                            docker stop nginx-website || true
                            docker rm nginx-website || true
                        fi
                    '''
                    
                    // Run the Flask app container in detached mode
                    sh 'docker run -d --network host -p 80:80 --name nginx-website ghcr.io/rayhanardiansyah17/cicdtest:latest'
                    
                    // Wait for Flask app to start up and be ready
                    sleep 5 // Increase wait time for Flask app startup
                    
                    sh 'docker ps -a'
                    sleep 2
                    // Check if the Flask app returns a 200 status code
                    script {
                        try {
                            // Check the status code and return an error if it's not 200
                            sh '''
                                STATUS=$(curl -o /dev/null -s -w "%{http_code}" http://host.docker.internal:80/)
                                if [ "$STATUS" -ne 200 ]; then
                                    echo "Nginx app returned status code $STATUS"
                                    exit 1
                                else
                                    echo "Nginx app returned status code 200"
                                fi
                            '''
                        } catch (Exception e) {
                            // Print the Nginx app logs if the test fails
                            sh 'docker logs nginx-website'
                            error('Nginx app did not return a 200 status code')
                        }
                    }
                }
            }
        }


        stage('Login to GHCR') {
            steps {
                script {
                    withCredentials([string(credentialsId: DOCKER_CREDENTIALS_ID, variable: 'PAT_CERT')]) {
                        sh 'echo $PAT_CERT | docker login ghcr.io -u rayhanardiansyah17 --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Image to GHCR') {
            steps {
                script {
                    sh 'docker push ghcr.io/rayhanardiansyah17/cicdtest:latest'
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Stop and remove the Flask app container
                    sh 'docker stop nginx-website || true'
                    sh 'docker rm nginx-website || true'
                    
                    // Optional: Remove the local Docker image
                    sh 'docker rmi ghcr.io/rayhanardiansyah17/cicdtest:latest || true'
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
