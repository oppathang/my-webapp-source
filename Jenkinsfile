pipeline {
    // 1. Cấu hình Pod Agent trên K8s có chứa container Docker (DinD)
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              # Container này chứa Docker daemon để build image
              - name: docker
                image: docker:24.0.5-dind
                securityContext:
                  privileged: true # Bắt buộc phải có để chạy được Docker trong Docker
                env:
                - name: DOCKER_TLS_CERTDIR
                  value: ""
              # Container jnlp mặc định của Jenkins (dùng để chạy các lệnh git)
              - name: jnlp
                image: jenkins/inbound-agent:latest
            '''
        }
    }
    
    environment {
        // Thông tin tài khoản Docker Hub
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-login')
        DOCKER_IMAGE = "oppathang/my-webapp-source"
        TAG = "v_${BUILD_NUMBER}"
        
        // Khai báo cho Docker client biết nơi chứa Docker daemon để kết nối
        DOCKER_HOST = "tcp://localhost:2375"
    }
    
    stages {
        stage('Build & Push Docker Image') {
            steps {
                // 2. Ép stage này phải chạy bên trong container tên là 'docker'
                container('docker') {
                    script {
                        sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                        sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${TAG}"
                    }
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                // 3. Sử dụng container mặc định 'jnlp' vì nó đã có sẵn công cụ Git
                container('jnlp') {
                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        script {
                            // Nhớ thay link này thành repo manifests thật của bạn
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
