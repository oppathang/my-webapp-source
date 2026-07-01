pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-cred')
        DOCKER_IMAGE = "<TÊN_DOCKER_HUB_CỦA_BẠN>/my-webapp"
        TAG = "v_${BUILD_NUMBER}"
    }
    stages {
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Xây dựng Image
                    sh "docker build -t ${DOCKER_IMAGE}:${TAG} ."
                    
                    // Đăng nhập và Push lên Docker Hub
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${TAG}"
                }
            }
        }
        stage('Update K8s Manifest') {
            steps {
                script {
                    // Jenkins clone repo manifests về, đổi version image và push lại lên GitLab
                    sh '''
                    git clone https://gitlab.com/<username-cua-ban>/my-webapp-manifests.git
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
