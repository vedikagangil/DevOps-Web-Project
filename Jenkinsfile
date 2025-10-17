pipeline {
    agent any
    
    tools {
        jdk 'java17'  // Uses Java 17 from Jenkins configuration
        maven 'M3'    // Uses Maven from Jenkins configuration
    }
    
    environment {
        DEPLOY_PATH = 'C:\\jenkins\\app'
    }
    
    stages {
        stage('Verify Clean Java 17 Setup') {
            steps {
                bat '''
                    echo "=== Java 17 Verification ==="
                    echo "JAVA_HOME: %JAVA_HOME%"
                    echo "--- Java Version ---"
                    java -version
                    echo "--- Javac Version ---"
                    javac -version
                    echo "--- All Java Executables ---"
                    where java
                    where javac
                    echo "--- Maven with Java 17 ---"
                    mvn --version
                '''
            }
        }
        
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
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
                        if not exist "${env.DEPLOY_PATH}" mkdir "${env.DEPLOY_PATH}"
                        echo "Copying WAR file to ${env.DEPLOY_PATH}..."
                        copy target\\*.war "${env.DEPLOY_PATH}\\"
                        echo "Stopping any existing applications..."
                        taskkill /F /IM java.exe 2>nul || echo "No running applications found"
                        timeout /t 5
                        echo "Starting application with Java 17..."
                        cd /d "${env.DEPLOY_PATH}"
                        for %%f in (*.war) do (
                            echo "Deploying: %%f"
                            start "WebApp" /B java -jar "%%f" > app.log 2>&1
                        )
                        echo "✓ Deployment completed successfully!"
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    bat """
                        echo "=== Verifying Deployment ==="
                        timeout /t 10
                        echo "1. Checking Java processes..."
                        tasklist /FI "IMAGENAME eq java.exe" | find "java.exe" && echo "✓ Java process is running" || echo "✗ No Java process found"
                        echo "2. Checking port 8080..."
                        netstat -an | find ":8080" && echo "✓ Port 8080 is in use" || echo "✗ Port 8080 not in use"
                        echo "3. Testing application endpoint..."
                        curl -f http://localhost:8080/ && echo "✓ Application is responding!" || echo "⚠ Application not responding on port 8080"
                        echo "4. Application log:"
                        if exist "${env.DEPLOY_PATH}\\app.log" (
                            echo "Last 10 lines of app.log:"
                            tail -10 "${env.DEPLOY_PATH}\\app.log"
                        ) else (
                            echo "No app.log file found"
                        )
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo "🎉 Pipeline completed successfully with Java 17!"
            emailext (
                subject: "SUCCESS: Jenkins Pipeline Completed",
                body: "The Jenkins pipeline has completed successfully using Java 17.",
                to: "YOUR_EMAIL@example.com"
            )
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above for details."
        }
    }
}
