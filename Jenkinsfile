pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'https://github.com/NautiluX/s3e'
        PROJECT_NAMESPACE = 'arcade'
        PROJECT_NAME = 'highscore'
        BUILD_CONFIG = 'buildconfig.yaml'
        IMAGE_STREAM = 'imagestream.yaml'
        DEPLOYMENT_CONFIG = 'deployment.yaml'
        SERVICE_CONFIG = 'service.yaml'
        WORKSPACE_DIR = "${env.WORKSPACE}/src"
        GIT_CREDENTIALS_ID = 'github-token' // The ID of the GitHub token credentials
    }

    stages {
        stage('Clean up repo') {
          steps {
            script {
              // cleanup repo
              sh '''
              rm -rf ${WORKSPACE_DIR}
              '''
            }
          }
        }

        stage('Prepare Workspace') {
            steps {
                script {
                    // Check if workspace directory exists and create it if not
                    sh '''
                    if [ ! -d "${WORKSPACE_DIR}" ]; then
                      mkdir -p ${WORKSPACE_DIR}
                      chmod 777 ${WORKSPACE_DIR}
                    fi
                    '''
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    // Clone the repository
                        sh '''
                        git clone https://$GITHUB_TOKEN@github.com/NautiluX/s3e ${WORKSPACE_DIR}
                       '''
                }
            }
        }

        stage('Configure Project') {
            steps {
                script {
                    // Configure the project by applying necessary configurations
                    sh '''
                    cd ${WORKSPACE_DIR}/highscore/ci
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

        stage('Wait for Build') {
            steps {
                script {
                    // Wait for the build to complete
                    timeout(time: 15, unit: 'MINUTES') {
                        waitUntil {
                            script {
                                def buildStatus = sh(script: "oc get builds -n ${PROJECT_NAMESPACE} -o jsonpath='{.items[?(@.metadata.annotations.openshift\\.io/build\\.name==\"${PROJECT_NAME}-1\")].status.phase}'", returnStdout: true).trim()
                                return buildStatus == 'Complete'
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy Project') {
            steps {
                script {
                    // Deploy the project
                    sh '''
                    cd ${WORKSPACE_DIR}/highscore/ci
                    oc apply -f ${DEPLOYMENT_CONFIG}
                    BUILDDATE=$(date +%Y-%m-%d-%H%M%S)
                    oc patch deployment ${PROJECT_NAME} -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"last-build\":\"$BUILDDATE\"}}}}}"
                    oc apply -f ${SERVICE_CONFIG}

                    # Generate and apply route
                    HOSTNAME="arcade.$(oc get ingresses.config.openshift.io cluster -o jsonpath='{.spec.domain}')"
                    cat <<EOF | oc apply -f -
                    apiVersion: route.openshift.io/v1
                    kind: Route
                    metadata:
                      labels:
                        app: ${PROJECT_NAME}
                      name: ${PROJECT_NAME}
                      namespace: ${PROJECT_NAMESPACE}
                    spec:
                      host: $HOSTNAME
                      path: /highscore
                      port:
                        targetPort: 8080
                      to:
                        kind: Service
                        name: ${PROJECT_NAME}
                    EOF
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
