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
        
        stage('Deploy to Tomcat') {
            steps {
                script {
                    bat """
                        echo "=== Tomcat Deployment on Port ${env.TOMCAT_PORT} ==="
                        echo "CATALINA_HOME: %CATALINA_HOME%"
                        
                        echo "1. Stopping Tomcat..."
                        call "%CATALINA_HOME%\\bin\\shutdown.bat" 2>nul || echo "Tomcat was not running"
                        
                        echo "2. Waiting for shutdown..."
                        ping -n 5 127.0.0.1 >nul
                        
                        echo "3. Cleaning old deployment..."
                        if exist "%CATALINA_HOME%\\webapps\\nextwork-web-project.war" (
                            del "%CATALINA_HOME%\\webapps\\nextwork-web-project.war"
                        )
                        if exist "%CATALINA_HOME%\\webapps\\nextwork-web-project" (
                            rmdir /S /Q "%CATALINA_HOME%\\webapps\\nextwork-web-project"
                        )
                        
                        echo "4. Deploying new WAR file..."
                        copy target\\nextwork-web-project.war "%CATALINA_HOME%\\webapps\\"
                        
                        echo "5. Starting Tomcat..."
                        start "Tomcat Server" /B "%CATALINA_HOME%\\bin\\startup.bat"
                        
                        echo "✓ Deployment completed! Tomcat starting on port ${env.TOMCAT_PORT}..."
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    bat """
                        echo "=== Verification on Port ${env.TOMCAT_PORT} ==="
                        echo "Waiting for Tomcat to start..."
                        ping -n 30 127.0.0.1 >nul
                        
                        echo "1. Java processes:"
                        tasklist /FI "IMAGENAME eq java.exe"
                        
                        echo "2. Testing Tomcat on port ${env.TOMCAT_PORT}..."
                        curl -f http://localhost:${env.TOMCAT_PORT}/ && echo "✓ Tomcat is running on port ${env.TOMCAT_PORT}!" || echo "Tomcat not responding on port ${env.TOMCAT_PORT}"
                        
                        echo "3. Testing application..."
                        curl -f http://localhost:${env.TOMCAT_PORT}/nextwork-web-project/ && echo "✓ Application deployed successfully!" || echo "Application endpoint not accessible"
                        
                        echo "4. Available endpoints:"
                        curl http://localhost:${env.TOMCAT_PORT}/nextwork-web-project/ || echo "Root context not accessible"
                        curl http://localhost:${env.TOMCAT_PORT}/manager/ || echo "Manager not accessible"
                        curl http://localhost:${env.TOMCAT_PORT}/host-manager/ || echo "Host manager not accessible"
                        
                        echo "5. Tomcat logs:"
                        if exist "%CATALINA_HOME%\\logs\\catalina.out" (
                            echo "Last 15 lines:"
                            tail -15 "%CATALINA_HOME%\\logs\\catalina.out"
                        ) else (
                            echo "Latest log file:"
                            for /f %%i in ('dir "%CATALINA_HOME%\\logs\\catalina.*.log" /B /O-D 2^>nul ^| head -1') do (
                                echo "File: %%i"
                                tail -15 "%CATALINA_HOME%\\logs\\%%i"
                            )
                        )
                    """
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: '**/*.war', allowEmptyArchive: true
            cleanWs()
        }
        success {
            echo "🎉 Deployment successful! Application available at: http://localhost:8081/nextwork-web-project/"
        }
    }
}
