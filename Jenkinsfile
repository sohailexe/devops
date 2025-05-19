pipeline {
    agent any // Runs on any available agent/node (ensure Docker & Docker Compose are on it)

    environment {
        // Define environment variables to be used in the pipeline
        COMPOSE_PROJECT_NAME = "devops" // Changed to be a bit more specific
        COMPOSE_FILE_PATH = "docker-compose.yaml" // Using your actual filename
        APP_RUN_DURATION_SECONDS = 30 // How long the app should run
    }

    stages {
        stage('Clean Workspace') {
            steps {
                // Clean up the workspace from previous builds
                deleteDir()
                echo "Workspace cleaned."
            }
        }

        stage('Checkout Code') {
            steps {
                // Fetches code from GitHub (configured in Jenkins job)
                checkout scm
                echo "Code checked out."
                // Verify that env files exist, otherwise docker-compose will fail
                sh 'ls -l env/' // Check if env directory and files are present
            }
        }

        stage('Build and Run Application') {
            steps {
                script {
                    try {
                        echo "Ensuring any previous run with project name '${env.COMPOSE_PROJECT_NAME}' is stopped..."
                        // Ensure any previous run with the same project name is stopped and removed
                        // The `|| true` ensures the command doesn't fail the build if containers aren't running
                        sh "docker-compose -p ${env.COMPOSE_PROJECT_NAME} -f ${env.COMPOSE_FILE_PATH} down --volumes --remove-orphans || true"

                        echo "Building and starting application using docker-compose..."
                        // The -d flag runs containers in detached mode (in the background)
                        // --build forces a rebuild of images if Dockerfile or context changed
                        sh "docker-compose -p ${env.COMPOSE_PROJECT_NAME} -f ${env.COMPOSE_FILE_PATH} up --build -d"

                        echo "Application services started for project '${env.COMPOSE_PROJECT_NAME}'."
                        echo "Application will run for ${env.APP_RUN_DURATION_SECONDS} seconds..."

                        // Wait for the specified duration
                        sleep time: env.APP_RUN_DURATION_SECONDS.toInteger(), unit: 'SECONDS'

                        echo "Time up! Application run duration reached."

                    } catch (e) {
                        echo "An error occurred during 'Build and Run Application' stage: ${e.getMessage()}"
                        currentBuild.result = 'FAILURE' // Mark build as failed
                        // We still want to try and cleanup in the post action
                        throw e // Re-throw the exception to properly fail the stage
                    }
                }
            }
        }
    }

    post {
        always {
            // This block runs regardless of the pipeline's success or failure
            echo "Pipeline finished. Cleaning up docker-compose services for project '${env.COMPOSE_PROJECT_NAME}'..."
            // Stop and remove containers, networks, and volumes associated with the project
            // The `|| true` ensures the command doesn't fail the build if it was already cleaned up or never started
            sh "docker-compose -p ${env.COMPOSE_PROJECT_NAME} -f ${env.COMPOSE_FILE_PATH} down --volumes --remove-orphans || true"
            echo "Cleanup complete."
        }
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check console output for details."
        }
    }
}