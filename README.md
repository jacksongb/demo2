# Python Web Demo

简单的 Flask Web 应用，用于 Jenkins 构建演示。

## 本地运行

```bash
pip install -r requirements.txt
python app.py
```

访问: http://localhost:5000

## Docker 构建

```bash
docker build -t python-web-demo .
docker run -p 5000:5000 python-web-demo
```

## Jenkins Pipeline 示例

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t python-web-demo:${BUILD_NUMBER} .'
            }
        }
        stage('Push') {
            steps {
                // 推送到镜像仓库
                // sh 'docker push your-registry/python-web-demo:${BUILD_NUMBER}'
            }
        }
    }
}
```
