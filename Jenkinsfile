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
                        echo "Checking if Tomcat is running..."
                        tasklist /FI "IMAGENAME eq java.exe" | find "java.exe" && echo "Tomcat is running" || echo "Tomcat is not running"
                        
                        echo "Graceful shutdown..."
                        call "%CATALINA_HOME%\\bin\\shutdown.bat" 2>nul || echo "Shutdown command sent"
                        
                        echo "Waiting for shutdown..."
                        ping -n 10 127.0.0.1 >nul
                        
                        echo "Force stop if still running..."
                        tasklist /FI "IMAGENAME eq java.exe" | find "java.exe" && (
                            echo "Tomcat still running, forcing stop..."
                            taskkill /F /IM java.exe 2>nul || echo "No Java processes to kill"
                        ) || echo "Tomcat stopped successfully"
                    """
                }
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                script {
                    bat """
                        echo "=== Deploying WAR File ==="
                        echo "Cleaning old deployment..."
                        if exist "%CATALINA_HOME%\\webapps\\nextwork-web-project.war" (
                            del "%CATALINA_HOME%\\webapps\\nextwork-web-project.war"
                        )
                        if exist "%CATALINA_HOME%\\webapps\\nextwork-web-project" (
                            rmdir /S /Q "%CATALINA_HOME%\\webapps\\nextwork-web-project"
                        )
                        
                        echo "Deploying new WAR file..."
                        copy target\\nextwork-web-project.war "%CATALINA_HOME%\\webapps\\"
                        echo "WAR file deployed to Tomcat"
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
                        start "Tomcat Server" /B "%CATALINA_HOME%\\bin\\startup.bat"
                        
                        echo "Waiting for startup..."
                        ping -n 30 127.0.0.1 >nul
                        echo "Tomcat startup completed"
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    bat """
                        echo "=== Final Verification ==="
                        echo "Checking Java processes..."
                        tasklist /FI "IMAGENAME eq java.exe"
                        
                        echo "Testing Tomcat on port ${env.TOMCAT_PORT}..."
                        curl -f http://localhost:${env.TOMCAT_PORT}/ && echo "SUCCESS: Tomcat is running!" || echo "Tomcat not responding"
                        
                        echo "Testing your application..."
                        curl -f http://localhost:${env.TOMCAT_PORT}/nextwork-web-project/ && echo "SUCCESS: Application is working!" || echo "Application not accessible"
                        
                        echo "Deployment directory:"
                        dir "%CATALINA_HOME%\\webapps\\"
                    """
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: '**/*.war', allowEmptyArchive: true
        }
        success {
            echo "🎉 SUCCESS! Your Jenkins CI/CD pipeline is COMPLETE!"
            echo "Your application is LIVE at: http://localhost:8081/nextwork-web-project/"
        }
    }
}
