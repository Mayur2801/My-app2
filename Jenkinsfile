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
                    // Use Jenkins build number for versioning: v1, v2, v3, ...
                    env.VERSION = "v${env.BUILD_NUMBER}"
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

sudo docker pull $DOCKER_IMAGE:$VERSION
sudo docker stop myapp || true
sudo docker rm myapp || true
sudo docker run -d --name myapp -p 80:80 $DOCKER_IMAGE:$VERSION
EOF
                    """
                }
            }
        }
    }
}
