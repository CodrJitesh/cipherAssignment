pipeline {
    agent any

    environment {
        // ── CHANGE THIS to your Docker Hub username ──
        DOCKERHUB_USER = 'jitesh'
        // ─────────────────────────────────────────────

        IMAGE = "${DOCKERHUB_USER}/simple-web"
        TAG   = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE}:${TAG} ."
            }
        }

        stage('Health Check') {
            steps {
                sh """
                    docker run -d --name health-test -p 8090:80 ${IMAGE}:${TAG}
                    sleep 3
                    curl -f http://localhost:8090
                    docker stop health-test && docker rm health-test
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push ${IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh """
                    docker stop webapp 2>/dev/null || true
                    docker rm   webapp 2>/dev/null || true
                    docker run -d --name webapp -p 3000:80 ${IMAGE}:${TAG}
                """
            }
        }
    }

    post {
        always {
            sh "docker rmi ${IMAGE}:${TAG} || true"
            sh "docker logout || true"
        }
        failure {
            sh "docker stop health-test 2>/dev/null || true"
            sh "docker rm   health-test 2>/dev/null || true"
        }
    }
}
