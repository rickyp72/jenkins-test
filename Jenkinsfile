pipeline {
    agent any

    environment {
        // Define any environment variables here if needed
        GIT_REPO_URL = 'https://github.com/NautiluX/s3e'
        PROJECT_NAMESPACE = 'arcade'
        PROJECT_NAME = 'highscore'
        BUILD_CONFIG = 'buildconfig.yaml'
        IMAGE_STREAM = 'imagestream.yaml'
        DEPLOYMENT_CONFIG = 'deployment.yaml'
        SERVICE_CONFIG = 'service.yaml'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    // Clone the repository
                    sh '''
                    git clone ${GIT_REPO_URL} /workspace/src
                    '''
                }
            }
        }

        stage('Configure Project') {
            steps {
                script {
                    // Configure the project by applying necessary configurations
                    sh '''
                    cd /workspace/src/highscore/ci
                    oc apply -f ${IMAGE_STREAM}
                    oc apply -f ${BUILD_CONFIG}
                    '''
                }
            }
        }

        stage('Build Project') {
            steps {
                script {
                    // Build the project
                    sh '''
                    oc start-build -F ${PROJECT_NAME} -n ${PROJECT_NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        always {
            // Cleanup or post actions can be added here
            cleanWs()
        }
    }
}
