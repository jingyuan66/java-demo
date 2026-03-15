pipeline {
    agent any
     tools {
        maven 'Maven 3.9.5'  // 'maven 3.9.5' 必须和你配置的 Name 完全一致
    }

    // 定义全局变量，方便修改
    environment {
        HTTP_PROXY = 'http://127.0.0.1:7890'
        HTTPS_PROXY = 'http://127.0.0.1:7890'
        // Git 仓库地址
        REPO_URL = 'https://github.com/jingyuan66/java-demo.git'
        // 分支名称
        BRANCH = 'main'
        // Maven 构建命令
        MAVEN_CMD = 'clean package'
        // 打包后的 JAR 包名（根据你的 pom.xml 定义）
        JAR_NAME = 'target/java-demo.jar'
        // 部署服务器的 IP 或域名
        DEPLOY_SERVER = '192.168.71.58'
        // 部署服务器的用户名
        DEPLOY_USER = 'zhang'
        // 部署路径
        DEPLOY_PATH = '/Users/zhang/Desktop/'
        // JAR 启动的文件名（复制到服务器后的名称）
        APP_NAME = 'java-demo.jar'
    }

    stages {
        stage('拉取代码') {
            steps {
                echo '正在从 Git 拉取代码...'
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Maven 构建') {
            steps {
                echo '正在使用 Maven 构建项目...'
                // 确保 Jenkins 全局配置中配置了 Maven
                sh "mvn ${MAVEN_CMD}"
            }
        }

        stage('单元测试') {
            steps {
                echo '正在运行单元测试...'
                sh 'mvn test'
            }
            post {
                // 如果测试失败，标记构建为不稳定
                failure {
                    echo '单元测试失败，请检查代码！'
                }
            }
        }

        stage('打包与存档') {
            steps {
                echo '正在归档构建产物...'
                // 将构建好的 JAR 包保存到 Jenkins 构建记录中
                archiveArtifacts artifacts: "${JAR_NAME}", fingerprint: true
            }
        }

         stage('Deploy') {
            steps {
                script {
                    echo '开始部署...'

                    // 1. 先测试 SSH 连接
                    sh """
                        echo "测试 SSH 连接..."
                        nc -zv ${DEPLOY_SERVER} 22 || {
                            echo "错误: 无法连接到服务器 ${DEPLOY_SERVER}:22"
                            echo "请检查："
                            echo "1. 服务器是否开机"
                            echo "2. SSH 服务是否运行"
                            echo "3. 防火墙是否允许 22 端口"
                            exit 1
                        }
                    """

                    // 2. 使用正确的路径格式进行部署
                    sh """
                        echo "开始复制文件..."
                        cp ${JAR_NAME} \
                            ${DEPLOY_PATH}${APP_NAME}

                        if [ \$? -eq 0 ]; then
                            echo "文件复制成功！"
                        else
                            echo "文件复制失败！"
                            exit 1
                        fi
                    """
                }
            }
        }
    }

    post {
        success {
            echo '恭喜，构建并部署成功！'
            // 可选：发送邮件或钉钉通知
        }
        failure {
            echo '构建或部署失败，请查看日志。'
            // 可选：发送失败通知
        }
    }
}