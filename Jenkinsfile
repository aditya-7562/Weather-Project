pipeline {
    agent any  // This will use the Jenkins agent to run the jobs

    environment {
        DOCKER_IMAGE = 'weather-app'
        DOCKER_TAG = 'latest'
        DOCKER_USERNAME = 'aditya7652'
        DOCKER_REGISTRY = 'docker.io'
        GITHUB_REPO = 'https://github.com/aditya-7562/Weather-Project.git'
        GITHUB_TOKEN = credentials('GITHUB_TOKEN') // Ensure GitHub token is set as a Jenkins secret
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Test Docker Container') {
            steps {
                script {
                    try {
                        // Run container in detached mode for testing
                        sh 'docker run -d -p 8080:80 --name weather-app-container $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                        // Give container a moment to start
                        sh 'sleep 5'
                        // Simple test to check if container is running
                        sh 'docker ps | grep weather-app-container'
                    } finally {
                        // Always clean up the test container
                        sh 'docker stop weather-app-container || true'
                        sh 'docker rm weather-app-container || true'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-password', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        }

        stage('Deploy to GitHub Pages') {
            steps {
                script {
                    // Create temp directory for GitHub Pages deployment
                    sh 'mkdir -p gh-pages-deploy'
                    
                    // Copy web files to deployment directory
                    sh 'cp -r index.html style.css script.js images gh-pages-deploy/'
                    
                    dir('gh-pages-deploy') {
                        withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GH_TOKEN')]) {
                            sh '''
                                git init
                                git config user.email "aditya.12114424@lpu.in"
                                git config user.name "aditya-7562"
                                
                                git add .
                                git commit -m "Deploy weather app to GitHub Pages"
                                
                                # Force push to gh-pages branch
                                git remote add origin https://aditya-7562:${GH_TOKEN}@github.com/aditya-7562/Weather-Project.git
                                git push -f origin master:gh-pages
                            '''
                        }
                    }
                    
                    // Clean up
                    sh 'rm -rf gh-pages-deploy'
                }
            }
        }
    }

    post {
        always {
            // Clean up any stray containers
            sh 'docker ps -q --filter "name=weather-app-container" | xargs -r docker stop || true'
            sh 'docker ps -a -q --filter "name=weather-app-container" | xargs -r docker rm || true'
            
            // Clean up Docker build cache if needed
            sh 'docker image prune -f'
        }
        success {
            echo 'Pipeline completed successfully! 🚀'
            echo 'Docker image pushed to Docker Hub and website deployed to GitHub Pages.'
        }
        failure {
            echo 'Pipeline failed! Please check the logs for errors. ⚠️'
        }
    }
}
