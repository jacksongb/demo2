pipeline {
    agent any

    environment {
        // 修改为你的GitHub仓库地址
        GIT_REPO = 'https://github.com/jacksongb/demo2.git'
        // Docker镜像名称
        IMAGE_NAME = 'python-web-demo'
        // Docker镜像标签（使用构建号）
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
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
                    # 检查Python语法
                    python3 -m py_compile app.py || true
                    
                    # 可以添加更多代码检查工具
                    # pip install flake8
                    # flake8 app.py
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
                    # 启动容器并测试
                    docker run -d --name test-container -p 5000:5000 ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 5
                    
                    # 健康检查
                    curl -f http://localhost:5000/health || exit 1
                    
                    # 清理测试容器
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
            // 清理工作空间
            cleanWs()
        }
    }
}
