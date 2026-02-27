pipeline {
    agent any

    environment {
        IMAGE_NAME = "2022bcd0053harshada/mlops-lab6:latest"
        CONTAINER_NAME = "ml-test-container"
        PORT = "8000"
        REPO_URL = "github.com/2022bcd0053-harshada/Lab7.git"
        BRANCH = "main"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh "docker pull ${IMAGE_NAME}"
            }
        }

        stage('Run Container') {
            steps {
                sh """
                docker rm -f ${CONTAINER_NAME} || true
                docker run -d -p ${PORT}:8000 --name ${CONTAINER_NAME} ${IMAGE_NAME}
                """
            }
        }

        stage('Wait for API') {
            steps {
                sh "sleep 10"
            }
        }

        stage('Valid Input Test') {
            steps {
                script {
                    def response = sh(
                        script: """
                        curl -s -X POST http://host.docker.internal:${PORT}/predict \
                        -H "Content-Type: application/json" \
                        -d @test_inputs/valid.json
                        """,
                        returnStdout: true
                    )

                    echo "Valid Response: ${response}"
                    writeFile file: 'valid_output.txt', text: response

                    if (!response.contains("prediction")) {
                        error "Valid test failed!"
                    }
                }
            }
        }

        stage('Invalid Input Test') {
            steps {
                script {
                    def response = sh(
                        script: """
                        curl -s -X POST http://host.docker.internal:${PORT}/predict \
                        -H "Content-Type: application/json" \
                        -d @test_inputs/invalid.json
                        """,
                        returnStdout: true
                    )

                    echo "Invalid Response: ${response}"
                    writeFile file: 'invalid_output.txt', text: response

                    if (!response.toLowerCase().contains("error")) {
                        error "Invalid test failed!"
                    }
                }
            }
        }

        stage('Push Results to GitHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"

                    git add valid_output.txt invalid_output.txt
                    git commit -m "Add test results from Jenkins" || echo "No changes"

                    git push https://${GIT_USER}:${GIT_PASS}@${REPO_URL} HEAD:${BRANCH}
                    """
                }
            }
        }

        stage('Stop Container') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME} || true
                docker rm ${CONTAINER_NAME} || true
                """
            }
        }
    }
}