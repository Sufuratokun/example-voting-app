pipeline {
    agent any
    environment {
        // Достаем ключи (Jenkins автоматически создаст переменные DOCKER_CREDS_USR и DOCKER_CREDS_PSW)
        DOCKER_CREDS = credentials('dockerhub-credentials')
        
        // === ЗАМЕНИТЕ ЭТИ ДАННЫЕ НА СВОИ ===
        DOCKER_HUB_USER = "ваш_логин_docker_hub"
        GITHUB_USER = "ваш_логин_github"
        BRANCH = "master" 
    }
    stages {
        stage('Setup Kubectl') {
            steps {
                sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                sh 'chmod +x ./kubectl'
            }
        }
        stage('Create K8s Secret') {
            steps {
                // Передаем ключи от Docker Hub внутрь кластера
                sh '''
                ./kubectl delete secret docker-cred --ignore-not-found
                ./kubectl create secret docker-registry docker-cred \
                  --docker-server=https://index.docker.io/v1/ \
                  --docker-username=$DOCKER_CREDS_USR \
                  --docker-password=$DOCKER_CREDS_PSW
                '''
            }
        }
        stage('Build & Push Vote (Kaniko)') {
            steps {
                // Генерируем манифест временной задачи (Job) и запускаем сборку
                sh '''
                cat <<EOF > kaniko-vote.yaml
                apiVersion: batch/v1
                kind: Job
                metadata:
                  name: kaniko-build-vote
                spec:
                  template:
                    spec:
                      containers:
                      - name: kaniko
                        image: gcr.io/kaniko-project/executor:latest
                        args:
                        - "--context=https://github.com/$GITHUB_USER/example-voting-app.git#refs/heads/$BRANCH"
                        - "--context-sub-path=vote"
                        - "--destination=$DOCKER_HUB_USER/vote:v1"
                        volumeMounts:
                        - name: docker-config
                          mountPath: /kaniko/.docker/
                      restartPolicy: Never
                      volumes:
                      - name: docker-config
                        secret:
                          secretName: docker-cred
                          items:
                            - key: .dockerconfigjson
                              path: config.json
                EOF
                ./kubectl delete job kaniko-build-vote --ignore-not-found
                ./kubectl apply -f kaniko-vote.yaml
                
                echo "Сборка началась! Ждем завершения (это может занять пару минут)..."
                ./kubectl wait --for=condition=complete job/kaniko-build-vote --timeout=300s
                '''
            }
        }
        stage('Deploy to k3d') {
            steps {
                // Деплоим старые манифесты, чтобы пайплайн просто успешно завершился
                sh './kubectl apply -f k8s-specifications/'
            }
        }
    }
}
