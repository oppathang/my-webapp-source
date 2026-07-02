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
        // 1. Định nghĩa địa chỉ máy chủ Registry của GitLab
        REGISTRY_URL = "registry.gitlab.com"
        
        // 2. SỬA CHỖ NÀY: Thay <username-gitlab> và <ten-repo-code> bằng thông tin thật của bạn
        // Cấu trúc chuẩn của GitLab: registry.gitlab.com/username/ten-repo/ten-image
        DOCKER_IMAGE = "${REGISTRY_URL}/oppathang/my-webapp-source/my-webapp"
        
        TAG = "v_${BUILD_NUMBER}"
        DOCKER_HOST = "tcp://localhost:2375"
    }
    
    stages {
        stage('Build & Push to GitLab Registry') {
            steps {
                container('docker') {
                    script {
                        sleep time: 5, unit: 'SECONDS'
                        
                        // Đóng gói ứng dụng
                        sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                        
                        // Đăng nhập vào hệ thống Registry của GitLab bằng ID mới tạo
                        withCredentials([usernamePassword(credentialsId: 'gitlab-registry-login', passwordVariable: 'GITLAB_PASS', usernameVariable: 'GITLAB_USER')]) {
                            sh "echo \"${GITLAB_PASS}\" | docker login ${REGISTRY_URL} -u \"${GITLAB_USER}\" --password-stdin"
                        }
                        
                        // Đẩy Image lên kho lưu trữ của GitLab
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
                            // Tải repo cấu hình manifest về để cập nhật
                            sh '''
                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/oppathang/my-webapp-manifests.git
                            cd my-webapp-manifests
                            
                            # Cập nhật đường dẫn Image mới (lúc này đã thành registry.gitlab.com/...)
                            sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${TAG}|g" deployment.yaml
                            
                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@example.com"
                            
                            git add deployment.yaml
                            git commit -m "Update image to GitLab Registry ${TAG}"
                            git push origin main
                            '''
                        }
                    }
                }
            }
        }
    }
}
