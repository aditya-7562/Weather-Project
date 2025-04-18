pipeline {
    agent any  // This will use the Jenkins master container itself to run the jobs

    environment {
        DOCKER_IMAGE = 'weather-app'
        DOCKER_TAG = 'latest'
        DOCKER_USERNAME = 'aditya7562'
        DOCKER_REGISTRY = 'docker.io'
        GITHUB_REPO = 'https://github.com/aditya-7562/Weather-Project.git'
        GITHUB_TOKEN = credentials('GITHUB_TOKEN') // Ensure GitHub token is set as a Jenkins secret
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git branch: 'main', url: "$GITHUB_REPO"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Ensure we have the right environment to run docker commands
                    node {
                        sh 'docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG .'
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    node {
                        sh 'docker run -d -p 8080:80 --name weather-app-container $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    node {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-password', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                            sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                            sh 'docker push $DOCKER_USERNAME/$DOCKER_IMAGE:$DOCKER_TAG'
                        }
                    }
                }
            }
        }

        stage('Deploy to GitHub Pages') {
            steps {
                script {
                    node {
                        sh '''
                            git config --global user.email "aditya.12114424@lpu.in"
                            git config --global user.name "aditya-7562"

                            # Clone gh-pages branch securely
                            git clone --depth=1 --branch=gh-pages https://x-access-token:$GITHUB_TOKEN@github.com/aditya-7562/Weather-Project.git gh-pages
                            cd gh-pages

                            rm -rf *

                            # Copy only static content
                            cp -r ../* .

                            # Remove files that shouldn't be deployed
                            rm -rf .gitignore Dockerfile Jenkinsfile README.md

                            git add .
                            git commit -m "Deploy updated weather app to GitHub Pages"
                            git push origin gh-pages
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                node {
                    sh '''
                        docker ps -q --filter "name=weather-app-container" | xargs -r docker stop || true
                        docker ps -a -q --filter "name=weather-app-container" | xargs -r docker rm || true
                    '''
                }
            }
        }
        success {
            echo 'Build completed successfully! 🚀'
        }
        failure {
            echo 'Build failed! Please check the logs for errors. ⚠️'
        }
    }
}
