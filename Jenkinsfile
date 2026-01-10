pipeline {
    agent any

    environment {
        // Injecting VITE_API_URL for the frontend to communicate with backend
        VITE_API_URL = "http://localhost:8000"
        EMAIL_TO = "dineshingale12@gmail.com"
    }

    stages {
        stage('Test') {
            steps {
                script {
                    try {
                        // Remove existing container if it exists (ignoring errors if not found)
                        try {
                            bat "docker rm -f test-runner"
                        } catch (Exception e) {}
                        
                        // Build the Docker image
                        bat "docker build -t test-runner ."
                        
                        // Run the container
                        try {
                            bat "docker run --name test-runner -e VITE_API_URL=${VITE_API_URL} test-runner"
                        } catch (Exception e) {
                            echo "Tests failed. marking build as FAILURE but proceeding to extract artifacts."
                            currentBuild.result = 'FAILURE'
                            throw e 
                        }
                    } finally {
                         // Extract test results (always try)
                         try {
                              bat "docker cp test-runner:/tmp/test-results.xml test-results.xml"
                         } catch (Exception e) {
                              echo "Failed to extract test results: ${e.getMessage()}"
                         }

                         // Extract screenshots (always try)
                         try {
                              bat "if not exist screenshots mkdir screenshots"
                              bat "docker cp test-runner:/tmp/screenshots/. screenshots/"
                         } catch (Exception e) {
                              echo "Failed to extract screenshots: ${e.getMessage()}"
                         }

                         // Cleanup
                         try {
                             bat "docker rm test-runner"
                         } catch (Exception e) {}
                    }
                }
            }
            post {
                always {
                    junit 'test-results.xml'
                    archiveArtifacts artifacts: 'screenshots/**/*.png', allowEmptyArchive: true
                }
            }
        }

        stage('Merge') {
            when {
                expression { return env.BRANCH_NAME != 'main' }
            }
            steps {
                script {
                    echo "Tests passed. Merging to main..."
                    bat "git checkout main"
                    bat "git merge ${env.BRANCH_NAME}"
                    bat "git push origin main"
                }
            }
        }
    }

    post {
        failure {
            emailext body: "Build Failed: ${env.BUILD_URL}",
                     subject: "Build Failed - ${env.JOB_NAME}",
                     to: "${EMAIL_TO}",
                     attachmentsPattern: 'screenshots/**/*.png'
        }
        success {
            emailext body: "Build Succeeded: ${env.BUILD_URL}",
                     subject: "Build Succeeded - ${env.JOB_NAME}",
                     to: "${EMAIL_TO}",
                     attachmentsPattern: 'screenshots/**/*.png'
        }
    }
}
