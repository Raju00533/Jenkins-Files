pipeline {
    agent any
    
    environment {
        APP_NAME = 'MY-APP'
        IMAGE_NAME = 'MY-APP-IMAGE'
        REGISTRY = 'docker.io'
        BRANCH_NAME = "${ENV.GIT_BRANCH}"
        BUILD_DIR = 'build'
        TEST_DIR = 'test'
        DEPLOY_ENV = 'staging'
        ARTIFACTS_DIR = 'build/artifacts'

    }

    options {
        discardOldBuilds(daysToKeep: 7, numToKeep: 10)
        timeout(time: 30, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    stages {
        stage ('checkout') {
            steps {
                checkout scm
            }
        }

        stage ('build') {
            steps {
                script {
                  echo "Building ${APP_NAME}..."
                  sh 'make build'
                }
            }
        }

        stage ('Test') {

            parallel {

                stage ('unit test') {

                    steps{
                        script{

                            echo "Running unit tests ..."

                            sh 'make unit-test'

                        }
                    }
                }

                stage ('Integration Tests') {
                    steps {

                        script {
                            echo "Running integration tests...."
                            sh 'make integration-test'

                        }
                    }
                }

                stage ('Lint') {

                    steps {
                        script {
                            echo "Running lint check"
                            sh 'make lint'

                        }
                    }
                }


            }
        }

        stage ('Docker build & Push') {
           when {
            branch 'main'

           }
           steps {
                script {
                    
                    echo "Building Docker image for ${IMAGE_NAME}..."
                    sh """
                        docker build -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG} .
                        docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_TAG}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    
                    echo "Deploying to ${DEPLOY_ENV}..."
                    sh "kubectl apply -f k8s/${DEPLOY_ENV}/deployment.yaml"
                }
            }
        }

        stage('Upload Artifacts') {
            steps {
                script {
                    
                    echo "Uploading artifacts..."
                    archiveArtifacts allowEmptyArchive: true, artifacts: '**/target/*.jar', followSymlinks: false
                    archiveArtifacts allowEmptyArchive: true, artifacts: '**/target/*.zip', followSymlinks: false
                }
            }
        }

        stage('Notify') {
            steps {
                script {
                    
                    if (currentBuild.result == 'SUCCESS') {
                        echo "Build completed successfully!"
                    } else {
                        echo "Build failed!"
                    }
                   
                    slackSend(channel: '#build-notifications', message: "Build ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
                }
            }
        }
    }

    post {
        success {
            echo 'Build succeeded!'
            
        }
        failure {
            echo 'Build failed!'
            
        }
        always {
            echo 'This will run after every build, regardless of success or failure.'
        }
    }
}