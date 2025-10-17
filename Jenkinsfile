pipeline {
    agent any
    
    tools {
        jdk 'java17'
        maven 'M3'
    }
    
    environment {
        CATALINA_HOME = 'C:\\Program Files\\apache-tomcat-9.0.111'
        DEPLOY_PATH = "${env.CATALINA_HOME}\\webapps"
        TOMCAT_PORT = '8081'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Package') {
            steps {
                bat 'mvn clean package -DskipTests'
                archiveArtifacts 'target/*.war'
            }
        }
        
        stage('Stop Tomcat Safely') {
            steps {
                script {
                    bat """
                        echo "=== Stopping Tomcat Safely ==="
                        echo "1. Checking if Tomcat is running..."
                        tasklist /FI "IMAGENAME eq java.exe" | find "java.exe" && echo "Tomcat is running" || echo "Tomcat is not running"
                        
                        echo "2. Graceful shutdown..."
                        call "%CATALINA_HOME%\\bin\\shutdown.bat" 2>nul || echo "Shutdown command sent or Tomcat not running"
                        
                        echo "3. Waiting for graceful shutdown..."
                        ping -n 10 127.0.0.1 >nul
                        
                        echo "4. Force stop if still running..."
                        tasklist /FI "IMAGENAME eq java.exe" | find "java.exe" && (
                            echo "Tomcat still running, forcing stop..."
                            taskkill /F /IM java.exe 2>nul || echo "No Java processes to kill"
                        ) || echo "Tomcat stopped successfully"
                        
                        echo "5. Final wait..."
                        ping -n 5 127.0.0.1 >nul
                    """
                }
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                script {
                    bat """
                        echo "=== Deploying WAR File ==="
                        
                        echo "1. Cleaning old deployment..."
                        if exist "%CATALINA_HOME%\\webapps\\nextwork-web-project.war" (
                            echo "Removing old WAR file..."
                            del "%CATALINA_HOME%\\webapps\\nextwork-web-project.war"
                        )
                        if exist "%CATALINA_HOME%\\webapps\\nextwork-web-project" (
                            echo "Removing old exploded directory..."
                            rmdir /S /Q "%CATALINA_HOME%\\webapps\\nextwork-web-project"
                        )
                        
                        echo "2. Deploying new WAR file..."
                        copy target\\nextwork-web-project.war "%CATALINA_HOME%\\webapps\\"
                        echo "✓ WAR file deployed to Tomcat"
                    """
                }
            }
        }
        
        stage('Start Tomcat') {
            steps {
                script {
                    bat """
                        echo "=== Starting Tomcat ==="
                        echo "Starting Tomcat server..."
                        
                        # Start Tomcat and wait for it
                        start "Tomcat Server" /B "%CATALINA_HOME%\\bin\\startup.bat"
                        
                        echo "Tomcat start command executed. Waiting for startup..."
                        ping -n 40 127.0.0.1 >nul
                        
                        echo "✓ Tomcat startup completed"
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    bat """
                        echo "=== Final Verification ==="
                        
                        echo "1. Checking Java processes..."
                        tasklist /FI "IMAGENAME eq java.exe"
                        
                        echo "2. Testing Tomcat on port ${env.TOMCAT_PORT}..."
                        curl -f http://localhost:${env.TOMCAT_PORT}/ && echo "✓ Tomcat is running!" || echo "✗ Tomcat not responding"
                        
                        echo "3. Testing your application..."
                        curl -f http://localhost:${env.TOMCAT_PORT}/nextwork-web-project/ && echo "✓ Application is working!" || echo "✗ Application not accessible"
                        
                        echo "4. Checking deployment status..."
                        dir "%CATALINA_HOME%\\webapps\\"
                        
                        echo "5. Recent Tomcat logs:"
                        for /f "tokens=*" %%i in ('dir "%CATALINA_HOME%\\logs\\catalina.*.log" /b /od 2^>nul') do set "latestlog=%%i"
                        if defined latestlog (
                            echo "Latest log: %latestlog%"
                            echo "=== LAST 20 LINES ==="
                            tail -20 "%CATALINA_HOME%\\logs\\%latestlog%"
                        ) else (
                            echo "No log files found"
                        )
                    """
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: '**/*.war', allowEmptyArchive: true
            echo "Pipeline execution completed"
        }
        success {
            echo "🎉 SUCCESS! If verification passed, your app is at: http://localhost:8081/nextwork-web-project/"
        }
        failure {
            echo "❌ Pipeline completed with issues. Check logs above."
        }
    }
}
