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
                            
                            # Крок 2: Виконуємо всі команди на віддаленому сервері як єдиний рядок
                            ssh -o StrictHostKeyChecking=no ubuntu@\${SERVER_IP} "pkill -f 'spring-boot-complete' || echo 'No process to kill'; cd ~/app && nohup java -jar *.jar > app.log 2>&1 </dev/null &"
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
                // Використовуємо синтаксис плагіна TelegramBot
                def message
                
                // Отримуємо всі необхідні дані з Credentials
                withCredentials([
                    string(credentialsId: 'telegram-chat-id', variable: 'CHAT_ID'),
                    string(credentialsId: 'ec2-server-ip', variable: 'SERVER_IP'),
                    string(credentialsId: 'telegram-bot-name', variable: 'BOT_NAME')
                ]) {
                    if (currentBuild.currentResult == 'SUCCESS') {
                        message = "✅ SUCCESS: Job ${env.JOB_NAME} [#${env.BUILD_NUMBER}] deployed successfully.\nApplication: http://\${SERVER_IP}:8080"
                    } else {
                        message = "❌ FAILED: Job ${env.JOB_NAME} [#${env.BUILD_NUMBER}] failed.\nLogs: ${env.BUILD_URL}"
                    }
                    // Відправляємо повідомлення через плагін TelegramBot, використовуючи змінну
                    telegramBot(bot: BOT_NAME, chatId: CHAT_ID, text: message)
                }
            }
        }
    }
}
