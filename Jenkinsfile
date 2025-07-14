pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/olhadevops/gs-spring-boot.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Deploy to EC2') {
            steps {
                // Отримуємо IP-адресу з Credentials
                withCredentials([string(credentialsId: 'ec2-server-ip', variable: 'SERVER_IP')]) {
                    // Використовуємо ключ доступу з Credentials
                    sshagent(credentials: ['ec2-ssh-key']) {
                        sh """
                            # Використовуємо змінну SERVER_IP для підключення
                            scp -o StrictHostKeyChecking=no target/*.jar ubuntu@${SERVER_IP}:~/app/
                            
                            ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} << EOF
                                pkill java || true
                                cd ~/app
                                nohup java -jar *.jar > app.log 2>&1 &
                            EOF
                        """
                    }
                }
            }
        }
    }
    // Блок для нотифікацій додамо на наступному кроці
    post {
        always {
            script {
                echo 'Pipeline finished.'
            }
        }
    }
}
