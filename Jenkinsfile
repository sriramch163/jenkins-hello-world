pipeline {
    agent any

    environment {
        BUILD_DIR = "build"
        GIT_REPO_URL = "https://github.com/sriramch163/jenkins-hello-world.git"

        IMAGE_NAME = "hello-world-app"
        CONTAINER_NAME = "hello-world-container"
    }

    parameters {
        string(
            name: 'GIT_BRANCH',
            defaultValue: 'main',
            description: 'Git Branch Name'
        )
    }

    stages {

        // -----------------------------------------------------------------------
        // Stage 1: Create Build Directory
        // -----------------------------------------------------------------------
        stage('Create Build Directory') {
            steps {
                sh """
                    mkdir -p "${WORKSPACE}/${BUILD_DIR}"
                """
            }
        }

        // -----------------------------------------------------------------------
        // Stage 2: Checkout Repository
        // -----------------------------------------------------------------------
        stage('Checkout') {
            steps {
                dir("${BUILD_DIR}") {
                    git url: "${env.GIT_REPO_URL}",
                        branch: "${params.GIT_BRANCH}"
                }
            }
        }

        // -----------------------------------------------------------------------
        // Stage 3: Build Docker Image
        // -----------------------------------------------------------------------
        stage('Build Docker Image') {
            steps {
                dir("${BUILD_DIR}") {
                    sh """
                        docker build -t ${IMAGE_NAME}:v1 .
                    """
                }
            }
        }

        // -----------------------------------------------------------------------
        // Stage 4: Stop Old Container
        // -----------------------------------------------------------------------
        stage('Stop Old Container') {
            steps {
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                """
            }
        }

        // -----------------------------------------------------------------------
        // Stage 5: Run Docker Container
        // -----------------------------------------------------------------------
        stage('Deploy Container') {
            steps {
                sh """
                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p 9000:80 \
                    ${IMAGE_NAME}:v1
                """
            }
        }

        // -----------------------------------------------------------------------
        // Stage 6: Verify Deployment
        // -----------------------------------------------------------------------
        stage('Verify Deployment') {
            steps {
                sh """
                    docker ps
                """
            }
        }

        // -----------------------------------------------------------------------
        // Stage 7: Push Image To DockerHub
        // -----------------------------------------------------------------------
        stage('Push Image To DockerHub') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag hello-world-app:v1 \
                        $DOCKER_USER/hello-world-app:v1

                        docker push \
                        $DOCKER_USER/hello-world-app:v1
                    '''
                }
            }
        }
    }

    post {

        success {
            echo 'Application deployed successfully 🚀'
        }

        failure {
            echo 'Pipeline failed ❌'
        }
    }
}
