piipeline {
    agent any

    environment {
        DOCKER_USERNAME = credentials('docker-username') // Jenkins credential ID for Docker Hub username
        DOCKER_PASSWORD = credentials('docker-password') // Jenkins credential ID for Docker Hub password
        KUBECONFIG = credentials('kubeconfig') // Jenkins secret text for Kubeconfig
        REPO_URL = 'https://github.com/ramya-012/wizdesk.git'
        DOCKER_IMAGE = "${env.DOCKER_USERNAME}/wizdesk:latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: env.REPO_URL
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        def customImage = docker.build(env.DOCKER_IMAGE)
                        customImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                    kubectl set image deployment/wizdesk wizdesk=${env.DOCKER_IMAGE}
                    kubectl rollout status deployment/wizdesk
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
