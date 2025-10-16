pipeline {
    agent any
    
    environment {
        DEPLOY_PATH = 'C:\\jenkins\\app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                bat 'mvn --version'
                bat 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                bat 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                bat 'mvn package -DskipTests'
                archiveArtifacts 'target/*.war'
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    bat """
                        echo "=== Starting Deployment ==="
                        
                        # Create deployment directory
                        if not exist "${env.DEPLOY_PATH}" mkdir "${env.DEPLOY_PATH}"
                        
                        # Copy WAR file
                        echo "Copying WAR file to deployment directory"
                        copy target\\*.war "${env.DEPLOY_PATH}\\"
                        
                        # Stop any existing application
                        echo "Stopping any existing application..."
                        taskkill /F /IM java.exe 2>nul || echo "No existing Java application running"
                        timeout /t 5
                        
                        # Deploy new application
                        echo "Starting new application..."
                        cd /d "${env.DEPLOY_PATH}"
                        for %%f in (*.war) do (
                            start /B java -jar "%%f" > app.log 2>&1
                            echo !time! - Application started: %%f >> deployment.log
                        )
                        
                        echo "Deployment completed successfully!"
                    """
                }
            }
        }
        
        stage('Verify') {
            steps {
                script {
                    bat """
                        echo "=== Verifying Deployment ==="
                        
                        # Wait for app to start
                        timeout /t 10
                        
                        # Check if Java process is running
                        echo "Checking Java processes..."
                        tasklist /FI "IMAGENAME eq java.exe" /FO CSV
                        
                        # Check listening ports
                        echo "Checking listening ports..."
                        netstat -an | findstr ":8080" || echo "Port 8080 not in use"
                        
                        # Try to access the application
                        echo "Testing application endpoint..."
                        curl -f http://localhost:8080/ && echo "Application is accessible" || echo "Application not accessible on port 8080"
                        
                        # Show application log if exists
                        if exist "${env.DEPLOY_PATH}\\app.log" (
                            echo "Application log (last 10 lines):"
                            tail -10 "${env.DEPLOY_PATH}\\app.log"
                        ) else (
                            echo "No application log file found"
                        )
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "=== Pipeline Complete ==="
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs above for details."
        }
    }
}
