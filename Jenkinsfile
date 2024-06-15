pipeline {

    agent any

    environment {
        NAME = 'flaskapp'
        IMAGE_VERSION = "${env.BUILD_NUMBER}"
        IMAGE_REPO = 'chinmayapradhan'
        SLACK_CHANNEL = '#'
    }

    stages {
        stage("Unit Tests") {
            steps {
                script {
                    echo "Implement unit tests if applicable..."
                    echo "This stage is a sample placeholder..."
                }
            }
        }
        stage("Build and Push Image") {
            steps {
                script {
                    echo "Building and Pushing to registry.."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION}"
                    }
                }
            }
        }
        stage("TRIVY scan") {
            steps {
                script {
                    sh "trivy image ${IMAGE_REPO}/${NAME}:${IMAGE_VERSION}"
                }
            }
        }
        stage("Update manifest and push") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                          git config --global user.name "chinmaya10000"
                          git config --global user.email "chinmayapradhan10000@gmail.com"
                          git remote set-url origin https://${GITHUB_TOKEN}@github.com/chinmaya10000/two-tier-flask-app.git
                          sed -i "s#chinmayapradhan.*#${IMAGE_REPO}/${NAME}:${IMAGE_VERSION}#g" eks/flaskapp.yaml
                          git add eks/flaskapp.yaml
                          git commit -m "Updated image version for Build - $IMAGE_VERSION"
                          git push origin HEAD:master
                        '''
                    }
                }
            }
        }
    }
    post {
        success {
            slackSend(channel: "${env.SLACK_CHANNEL}", message: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) completed successfully.")
        }
        failure {
            slackSend(channel: "${env.SLACK_CHANNEL}", message: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) failed.")
        }
    }
}