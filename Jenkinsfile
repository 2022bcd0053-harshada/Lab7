pipeline {
    agent any

    environment {
        IMAGE_NAME = "2022bcd0053harshada/mlops-lab6:latest"
        CONTAINER_NAME = "ml-test-container"
        PORT = "8000"
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
                        curl -s -X POST http://localhost:${PORT}/predict \
                        -H "Content-Type: application/json" \
                        -d @test_inputs/valid.json
                        """,
                        returnStdout: true
                    )

                    echo "Valid Response: ${response}"

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
                        curl -s -X POST http://localhost:${PORT}/predict \
                        -H "Content-Type: application/json" \
                        -d @test_inputs/invalid.json
                        """,
                        returnStdout: true
                    )

                    echo "Invalid Response: ${response}"

                    if (!response.toLowerCase().contains("error")) {
                        error "Invalid test failed!"
                    }
                }
            }
        }

        stage('Stop Container') {
            steps {
                sh """
                docker stop ${CONTAINER_NAME}
                docker rm ${CONTAINER_NAME}
                """
            }
        }
    }
}