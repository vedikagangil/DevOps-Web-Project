pipeline {
    agent any // This tells Jenkins to allocate an executor on any available agent (node)

    environment {
        // Define environment variables for your S3 bucket and application name
        S3_BUCKET = 'your-build-artifacts-bucket-name'
        APPLICATION_NAME = 'YourCodeDeployApplicationName'
        DEPLOYMENT_GROUP = 'YourCodeDeployDeploymentGroupName'
    }

    stages {
        stage('Checkout') {
            steps {
                // This replaces the GitHub source connection in CodePipeline
                git branch: 'master',
                    url: 'https://github.com/vedikagangil/DevOps-Web-Project.git'
            }
        }

        stage('Build and Unit Test') {
            steps {
                // This replaces the CodeBuild project
                sh 'mvn -f path/to/your/pom.xml clean compile test'
            }
            post {
                always {
                    // Always publish JUnit test results, even if the stage fails
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn -f path/to/your/pom.xml package -DskipTests'
                // The packaged WAR/JAR file is now in the target/ directory
            }
        }

        stage('Upload to S3') {
            steps {
                // This uses the AWS CLI (pre-installed on the Jenkins instance) to push the artifact
                // The AWS Steps plugin provides a more native 's3Upload' step as an alternative.
                sh """
                    aws s3 cp path/to/your/target/your-app.war \
                    s3://${env.S3_BUCKET}/build-artifacts/your-app-${env.BUILD_NUMBER}.war
                """
            }
        }

        stage('Deploy with CodeDeploy') {
            steps {
                // This replaces the CodeDeploy stage in CodePipeline.
                // It triggers an external CodeDeploy deployment.
                script {
                    def deploymentId = sh(
                        script: """
                            aws deploy create-deployment \
                                --application-name ${env.APPLICATION_NAME} \
                                --deployment-group-name ${env.DEPLOYMENT_GROUP} \
                                --s3-location bucket=${env.S3_BUCKET},key=build-artifacts/your-app-${env.BUILD_NUMBER}.war,bundleType=zip \
                                --query 'deploymentId' --output text
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Deployment triggered: $deploymentId"

                    // Optional: Wait for deployment to complete
                    sh """
                        aws deploy wait deployment-successful --deployment-id $deploymentId
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace after build, or send notifications
            cleanWs()
        }
        failure {
            // Send an email or Slack notification on failure
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                 body: "Check the build at: ${env.BUILD_URL}"
        }
    }
}
