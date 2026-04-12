pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        jdk 'jdk-17'
    }

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "54.198.143.91"
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

        stage('Copy JAR to EC2') {
            steps {
                bat """
                scp -o StrictHostKeyChecking=no target\\*.jar %EC2_USER%@%EC2_HOST%:/home/ubuntu/app/%APP_NAME%
                """
            }
        }

        stage('Deploy on EC2') {
            steps {
                bat """
                sbat -o StrictHostKeyChecking=no %EC2_USER%@%EC2_HOST% ^
                "pkill -f 'java -jar' || true && ^
                export WEATHER_API_KEY=%WEATHER_API_KEY% && ^
                nohup java -jar /home/ubuntu/app/%APP_NAME% > app.log 2>&1 &"
                """
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