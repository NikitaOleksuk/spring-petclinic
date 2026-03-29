pipeline {
    agent { label 'frankfurt' } 

    tools {
        jdk 'jdk17' 
    }

    environment {
        DOCKER_HUB_USER = 'nikitaoleksuk'
        DOCKER_HUB_CREDS = 'docker-hub-creds' 
        APP_NAME        = 'petclinic'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Jenkins сам подтянет нужную ветку (фичу или main)
                checkout scm
            }
        }

        stage('Unit Tests') {
            // Эта стадия работает ВСЕГДА и ДЛЯ ВСЕХ веток. 
            // Так ты проверяешь, что код разработчика не сломал тесты.
            steps {
                sh "chmod +x gradlew"
                sh "./gradlew test"
            }
        }

        stage('Build & Push Docker Image') {
            // Ограничиваем: собираем и пушим образ ТОЛЬКО если это ветка main
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_HUB_CREDS}") {
                        def customImage = docker.build("${DOCKER_HUB_USER}/${APP_NAME}")
                        customImage.push("${env.BUILD_NUMBER}")
                        customImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy Locally') {
            // Ограничиваем: деплоим ТОЛЬКО если это ветка main
            when {
                branch 'main'
            }
            steps {
                sh "docker-compose -p petclinic pull"
                sh "docker-compose -p petclinic up -d"
            }
        }
    }

    post {
        always {
            junit '**/build/test-results/test/*.xml'
        }
    }
}
