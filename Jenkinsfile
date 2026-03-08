pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/jacksongb/demo2.git'
        IMAGE_NAME = 'python-web-demo'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 拉取代码...'
                checkout scm
            }
        }

        stage('Code Quality Check') {
            steps {
                echo '🔍 检查代码...'
                sh '''
                    python3 -m py_compile app.py || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🏗️ 构建Docker镜像...'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest"
            }
        }

        stage('Test Image') {
            steps {
                echo '🧪 测试镜像...'
                sh '''
                    docker run -d --name test-container -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 5
                    curl -f http://localhost:5000/health || exit 1
                    docker stop test-container
                    docker rm test-container
                '''
            }
        }
    }

    post {
        success {
            echo '✅ 构建成功！'
            echo "镜像: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo '❌ 构建失败！'
        }
        always {
            cleanWs()
        }
    }
}
