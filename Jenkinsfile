pipeline {
    agent any
    
    environment {
        // 1. Gọi đúng ID credential 'dockerhub-log' bạn đã tạo trên Jenkins
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-login')
        
        // 2. Khai báo tên Image (Thay 'tài_khoản_docker_hub' bằng username Docker Hub của bạn)
        // Ví dụ: "oppathang/my-webapp-source"
        DOCKER_IMAGE = "oppathang/my-webapp-source"
        
        // Tạo tag động dựa trên số lần build của Jenkins
        TAG = "v_${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Đóng gói code từ GitHub thành Docker Image
                    sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                    
                    // Đăng nhập vào Docker Hub sử dụng credential 'dockerhub-log'
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    
                    // Đẩy Image lên Docker Hub
                    sh "docker push ${DOCKER_IMAGE}:${TAG}"
                }
            }
        }
        
        stage('Update K8s Manifest') {
            steps {
                script {
                    // Clone repo chứa cấu hình K8s (GitOps) từ GitHub về
                    // LƯU Ý: Đổi link bên dưới thành link repo chứa file deployment.yaml của bạn
                    sh '''
                    git clone https://github.com/oppathang/my-webapp-manifests.git
                    cd my-webapp-manifests
                    
                    # Tìm và thay thế dòng chứa 'image:' trong file deployment.yaml bằng image mới
                    sed -i "s|image:.*|image: ${DOCKER_IMAGE}:${TAG}|g" deployment.yaml
                    
                    # Cấu hình git user để commit
                    git config user.name "Jenkins CI"
                    git config user.email "jenkins@example.com"
                    
                    # Lưu thay đổi và đẩy lên lại GitHub
                    git add deployment.yaml
                    git commit -m "Update image to ${TAG}"
                    git push origin main
                    '''
                }
            }
        }
    }
}
