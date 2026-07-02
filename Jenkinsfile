pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: docker
                image: docker:24.0.5-dind
                # 1. Ép Jenkins chạy lệnh khởi động Docker Daemon, không được ghi đè
                command: ["dockerd-entrypoint.sh"]
                args: ["--tls=false"]
                securityContext:
                  privileged: true
                env:
                - name: DOCKER_TLS_CERTDIR
                  value: ""
              - name: jnlp
                image: jenkins/inbound-agent:latest
            '''
        }
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-login')
        DOCKER_IMAGE = "oppathang/my-webapp-source"
        TAG = "v_${BUILD_NUMBER}"
        DOCKER_HOST = "tcp://localhost:2375"
    }
    
    stages {
        stage('Build & Push Docker Image') {
            steps {
                container('docker') {
                    script {
                        // 2. Chờ 5 giây để Docker daemon khởi động hoàn toàn trước khi build
                        sleep time: 5, unit: 'SECONDS'
                        
                        sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                        sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${TAG}"
                    }
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                container('jnlp') {
                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        script {
                            // Nhớ thay link này thành link repo manifests thật của bạn
                            sh '''
                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/oppathang/my-webapp-manifests.git
                            cd my-webapp-manifests
                            
                            sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${TAG}|g" deployment.yaml
                            
                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@example.com"
                            
                            git add deployment.yaml
                            git commit -m "Update image to ${TAG}"
                            git push origin main
                            '''
                        }
                    }
                }
            }
        }
    }
}
