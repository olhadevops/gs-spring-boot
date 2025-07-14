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
                withCredentials([string(credentialsId: 'ec2-server-ip', variable: 'SERVER_IP')]) {
                    sshagent(credentials: ['ec2-ssh-key']) {
                        sh """
                            # 1. Копіюємо зібраний .jar файл на сервер
                            #    -o StrictHostKeyChecking=no - ігноруємо перевірку ключа хоста
                            scp -o StrictHostKeyChecking=no complete/target/*.jar ubuntu@${SERVER_IP}:~/app/
                            
                            # 2. Виконуємо скрипт розгортання на віддаленому сервері
                            ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} << EOF
                                # Вбиваємо старий процес, якщо він є
                                pkill -f 'spring-boot-complete' || echo "No process to kill"
                                
                                # Переходимо в папку і запускаємо новий .jar у фоновому режимі
                                # </dev/null & - гарантує, що процес повністю від'єднається від SSH-сесії
                                cd ~/app && nohup java -jar *.jar > app.log 2>&1 </dev/null &
                            EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
    }
}
