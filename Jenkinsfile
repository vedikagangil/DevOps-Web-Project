pipeline {
    agent any
    
    tools {
        jdk 'java17'  // This should now point to C:\Program Files\Java\jdk-17
        maven 'M3'
    }
    
    environment {
        DEPLOY_PATH = 'C:\\jenkins\\app'
    }
    
    stages {
        stage('Verify Java Configuration') {
            steps {
                bat '''
                    echo "=== Java Configuration Check ==="
                    echo "JAVA_HOME: %JAVA_HOME%"
                    echo "MAVEN_HOME: %MAVEN_HOME%"
                    echo "--- Checking Java Installation ---"
                    dir "%JAVA_HOME%" /AD && echo "✓ JAVA_HOME directory exists" || echo "✗ JAVA_HOME directory missing"
                    if exist "%JAVA_HOME%\\bin\\java.exe" (
                        echo "✓ java.exe found in JAVA_HOME\\bin"
                        "%JAVA_HOME%\\bin\\java.exe" -version
                    ) else (
                        echo "✗ java.exe NOT found in JAVA_HOME\\bin"
                    )
                    echo "--- Maven Check ---"
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
                        echo "Copying WAR files..."
                        copy target\\*.war "${env.DEPLOY_PATH}\\"
                        echo "Stopping existing applications..."
                        taskkill /F /IM java.exe 2>nul || echo "No existing Java applications found"
                        timeout /t 5
                        echo "Starting new application..."
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
                        echo "Checking Java processes..."
                        tasklist /FI "IMAGENAME eq java.exe" | find "java.exe" && echo "✓ Java process is running" || echo "✗ No Java process found"
                        echo "Checking port 8080..."
                        netstat -an | find ":8080" && echo "✓ Port 8080 is in use" || echo "✗ Port 8080 not in use"
                        echo "Testing application..."
                        curl -f http://localhost:8080/ && echo "✓ Application is responding!" || echo "⚠ Application not responding on port 8080"
                        if exist "${env.DEPLOY_PATH}\\app.log" (
                            echo "Application log (last 5 lines):"
                            tail -5 "${env.DEPLOY_PATH}\\app.log"
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
            echo "🎉 Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above."
        }
    }
}
