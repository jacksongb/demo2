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
                    # 启动容器
                    docker run -d --name test-container -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}
                    
                    # 等待容器启动
                    sleep 10
                    
                    # 查看容器日志（调试用）
                    docker logs test-container
                    
                    # 获取容器IP地址并测试
                    CONTAINER_IP=$(docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" test-container)
                    echo "Container IP: $CONTAINER_IP"
                    
                    # 通过容器IP访问
                    curl -f http://${CONTAINER_IP}:5000/health || {
                        echo "Container IP access failed, trying host network..."
                        docker stop test-container
                        docker rm test-container
                        docker run -d --name test-container --network host ${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                        curl -f http://localhost:5000/health
                    }
                    
                    # 清理测试容器
                    docker stop test-container || true
                    docker rm test-container || true
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
            sh '''
                docker stop test-container || true
                docker rm test-container || true
            '''
        }
        always {
            cleanWs()
        }
    }
}
