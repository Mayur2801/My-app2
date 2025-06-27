pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = 'mayur2808/myapp'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mayur2801/My-app2.git'
            }
        }

        stage('Set Version') {
            steps {
                script {
                    def shortCommit = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    def timestamp = sh(script: "date +%Y%m%d%H%M%S", returnStdout: true).trim()
                    env.VERSION = "${timestamp}-${shortCommit}"
                    echo "Generated version: ${env.VERSION}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$VERSION .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE:$VERSION
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@ec2-3-94-107-19.compute-1.amazonaws.com << 'EOF'
                        if ! command -v docker &> /dev/null; then
                            echo "Docker not found. Installing Docker..."

                            if [ -f /etc/os-release ]; then
                                . /etc/os-release
                                OS=\$ID
                            fi

                            if [ "\$OS" = "amzn" ]; then
                                sudo yum update -y
                                sudo yum install -y docker
                                sudo systemctl start docker
                                sudo systemctl enable docker
                            elif [ "\$OS" = "ubuntu" ]; then
                                sudo apt update -y
                                sudo apt install -y docker.io
                                sudo systemctl start docker
                                sudo systemctl enable docker
                            else
                                echo "Unsupported OS. Manual Docker install may be required."
                                exit 1
                            fi

                            sudo usermod -aG docker ec2-user
                            echo "Docker installed successfully."
                        else
                            echo "Docker is already installed."
                        fi

                        docker pull $DOCKER_IMAGE:$VERSION
                        docker stop myapp || true
                        docker rm myapp || true
                        docker run -d --name myapp -p 80:80 $DOCKER_IMAGE:$VERSION
                        EOF
                    """
                }
            }
        }
    }
}
