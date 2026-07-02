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
        // 1. Đường dẫn Registry của GitHub
        REGISTRY_URL = "ghcr.io"
        
        // Cấu trúc chuẩn của GitHub: ghcr.io/username/ten-image
        DOCKER_IMAGE = "${REGISTRY_URL}/oppathang/my-webapp"
        
        TAG = "v_${BUILD_NUMBER}"
        DOCKER_HOST = "tcp://localhost:2375"
    }
    
    stages {
        stage('Build & Push to GitHub Packages') {
            steps {
                container('docker') {
                    script {
                        sleep time: 5, unit: 'SECONDS'
                        
                        // Đóng gói ứng dụng
                        sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                        
                        // Sử dụng chính cái 'github-token' của bạn để đăng nhập vào ghcr.io
                        withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GH_TOKEN', usernameVariable: 'GH_USER')]) {
                            sh 'echo $GH_TOKEN | docker login ghcr.io -u $GH_USER --password-stdin'
                        }
                        
                        // Đẩy Image lên GitHub Packages
                        sh "docker push ${DOCKER_IMAGE}:${TAG}"
                    }
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                container('jnlp') {
                    // Sử dụng tiếp 'github-token' để clone và push code sửa đổi cấu hình K8s
                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        script {
                            sh '''
                            git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/oppathang/my-webapp-manifests.git
                            cd my-webapp-manifests
                            
                            # Sửa dòng chứa image sang đường dẫn ghcr.io mới
                            sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${TAG}|g" deployment.yaml
                            
                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@example.com"
                            
                            git add deployment.yaml
                            git commit -m "Update image to GHCR ${TAG}"
                            git push origin main
                            '''
                        }
                    }
                }
            }
        }
    }
}
