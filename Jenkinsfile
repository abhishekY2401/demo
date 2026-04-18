pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        jdk 'jdk-17'
    }

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "44.213.157.182"
        EC2_SSH_PORT = "22"
        APP_NAME = "app.jar"

        WEATHER_API_KEY = credentials('weather_api_key')
    }

    stages {
        
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/abhishekY2401/demo.git'
            }
        }

        stage('Build JAR') {
            steps {
                bat "mvn clean package -DskipTests"
            }
        }

        stage('Validate EC2 SSH Connectivity') {
            steps {
                bat '''
                powershell -NoProfile -ExecutionPolicy Bypass -Command "try { $ip = (Invoke-RestMethod -Uri 'https://checkip.amazonaws.com' -TimeoutSec 10).Trim(); Write-Host 'Jenkins public egress IP:' $ip } catch { Write-Host 'Could not determine Jenkins public egress IP automatically.' }"
                powershell -NoProfile -ExecutionPolicy Bypass -Command "$result = Test-NetConnection -ComputerName '%EC2_HOST%' -Port %EC2_SSH_PORT% -WarningAction SilentlyContinue; if (-not $result.TcpTestSucceeded) { Write-Host 'ERROR: Cannot reach %EC2_HOST% on port %EC2_SSH_PORT% from Jenkins agent.'; exit 1 } else { Write-Host 'SSH connectivity check passed.' }"
                '''
            }
        }

        stage('Copy JAR to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    bat """
                    icacls "%SSH_KEY%" /inheritance:r
                    icacls "%SSH_KEY%" /remove:g "BUILTIN\\Users" "NT AUTHORITY\\Authenticated Users" "Everyone"
                    icacls "%SSH_KEY%" /grant:r "%USERNAME%:R" "NT AUTHORITY\\SYSTEM:R" "BUILTIN\\Administrators:R"
                    scp -P %EC2_SSH_PORT% -o IdentitiesOnly=yes -o ConnectTimeout=15 -o ConnectionAttempts=2 -o StrictHostKeyChecking=no -i "%SSH_KEY%" target\\demo-0.0.1-SNAPSHOT.jar %SSH_USER%@%EC2_HOST%:/home/ubuntu/app/%APP_NAME%
                    """
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                    bat """
                    icacls "%SSH_KEY%" /inheritance:r
                    icacls "%SSH_KEY%" /remove:g "BUILTIN\\Users" "NT AUTHORITY\\Authenticated Users" "Everyone"
                    icacls "%SSH_KEY%" /grant:r "%USERNAME%:R" "NT AUTHORITY\\SYSTEM:R" "BUILTIN\\Administrators:R"
                    ssh -p %EC2_SSH_PORT% -o IdentitiesOnly=yes -o ConnectTimeout=15 -o ConnectionAttempts=2 -o StrictHostKeyChecking=no -i "%SSH_KEY%" %SSH_USER%@%EC2_HOST% "mkdir -p /home/ubuntu/app && pkill -f 'java -jar /home/ubuntu/app/%APP_NAME%' || true; WEATHER_API_KEY='%WEATHER_API_KEY%' nohup java -jar /home/ubuntu/app/%APP_NAME% >/home/ubuntu/app/app.log 2>&1 &"
                    """
                }    
            }
        }

        stage('Health Check') {
            steps {
                bat "timeout /t 10"
                bat "curl http://%EC2_HOST%:8081/weather/mumbai"
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}