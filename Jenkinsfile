@Library("shared") _

pipeline {
    agent { label "kshama" }

    environment {
        dockerHubUser = 'kshama614'
        dockerHubRepo = 'notes-app'
        imageVersion = "v1.0.${BUILD_NUMBER}"
        dockerHubImage = "${dockerHubUser}/${dockerHubRepo}:${imageVersion}"
    }

    stages {
        stage("Hello") {
            steps {
                script {
                    hello()
                }
            }
        }

        stage("Code") {
            steps {
                script {
                    clone('https://github.com/LondheShubham153/django-notes-app.git', 'main')
                }
            }
        }

        stage("Test") {
            steps {
                echo '🔍 Running Django unit tests...'
                sh 'pip install -r requirements.txt'
                sh 'python3 manage.py test'
            }
        }

        stage("Build") {
            steps {
                script {
                    docker_build(dockerHubRepo, imageVersion, dockerHubUser)
                }
            }
        }

        stage("Push to DockerHub") {
            steps {
                script {
                    docker_push(dockerHubRepo, imageVersion, dockerHubUser)
                }
            }
        }

        stage("Deploy") {
            steps {
                script {
                    echo '🚀 Deploying the application using Docker Compose...'
                    sh 'docker-compose down || true'
                    sh """
                        sed -i 's|image:.*|image: ${dockerHubImage}|' docker-compose.yml
                    """
                    sh 'docker-compose up -d'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment of ${dockerHubImage} completed successfully!"
        }
        failure {
            echo '❌ Deployment failed. Check the console logs for more info.'
        }
        cleanup {
            echo '🧹 Cleaning up unused Docker images...'
            sh 'docker image prune -f'
        }
    }
}
