pipeline {
    // 1. Агент, на якому буде виконуватися пайплайн
    agent any

    // 2. Інструменти, які потрібно підготувати
    tools {
        // Вказуємо точну назву Maven, як у Manage Jenkins -> Tools
        maven 'Maven 3.9.10'
    }

    // 3. Етапи виконання пайплайну
    stages {
        // Етап 1: Збірка проєкту
        stage('Build') {
            steps {
                // Переходимо в папку 'complete' і виконуємо збірку
                sh 'cd complete && mvn clean install'
            }
        }

        // Етап 2: Розгортання на сервері EC2
        stage('Deploy to EC2') {
            steps {
                // Використовуємо withCredentials для безпечного доступу до IP-адреси
                withCredentials([string(credentialsId: 'ec2-server-ip', variable: 'SERVER_IP')]) {
                    // Використовуємо sshagent для безпечного доступу до SSH-ключа
                    sshagent(credentials: ['ec2-ssh-key']) {
                        sh """
                            # Крок 1: Копіюємо зібраний .jar файл на сервер
                            scp -o StrictHostKeyChecking=no complete/target/*.jar ubuntu@\${SERVER_IP}:~/app/
                            
                            # Крок 2: Створюємо скрипт розгортання на віддаленому сервері
                            ssh ubuntu@\${SERVER_IP} 'cat > /home/ubuntu/deploy.sh' <<'END_OF_SCRIPT'
#!/bin/bash
echo "--> Stopping old process..."
pkill -f 'spring-boot-complete' || echo "No process to kill."
sleep 2
echo "--> Starting new process..."
cd /home/ubuntu/app
nohup java -jar *.jar > app.log 2>&1 &
echo "--> Deployment finished."
END_OF_SCRIPT

                            # Крок 3: Робимо скрипт виконуваним
                            ssh ubuntu@\${SERVER_IP} "chmod +x /home/ubuntu/deploy.sh"

                            # Крок 4: Запускаємо скрипт розгортання
                            ssh ubuntu@\${SERVER_IP} "/home/ubuntu/deploy.sh"
                        """
                    }
                }
            }
        }
    }

    // 4. Дії, які виконуються після завершення всіх етапів
    post {
        always {
            script {
                // Отримуємо всі необхідні дані з Credentials
                withCredentials([
                    string(credentialsId: 'telegram-chat-id', variable: 'CHAT_ID'),
                    string(credentialsId: 'ec2-server-ip', variable: 'SERVER_IP'),
                    string(credentialsId: 'telegram-bot-token', variable: 'BOT_TOKEN')
                ]) {
                    def message
                    if (currentBuild.currentResult == 'SUCCESS') {
                        message = "✅ SUCCESS: Job ${env.JOB_NAME} [#${env.BUILD_NUMBER}] deployed successfully.\\n\\nApplication: http://\${SERVER_IP}:8080"
                    } else {
                        message = "❌ FAILED: Job ${env.JOB_NAME} [#${env.BUILD_NUMBER}] failed.\\n\\nLogs: ${env.BUILD_URL}"
                    }
                    // Відправляємо повідомлення напряму через Telegram API за допомогою curl
                    sh """
                        curl -s -X POST https://api.telegram.org/bot\${BOT_TOKEN}/sendMessage -d chat_id=\${CHAT_ID} -d text="${message}"
                    """
                }
            }
        }
    }
}
