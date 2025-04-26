pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: '<your_git_credentials_id>', url: '<your_remote_repository_url>'
            }
        }
        stage('Build') {
            steps {
                sh 'python -m venv venv'
                sh 'source venv/bin/activate'
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'source venv/bin/activate'
                sh 'python -m unittest discover .' # Assuming you have tests
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("my-flask-app:${BUILD_ID}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('<your_dockerhub_registry>', '<your_dockerhub_credentials_id>') {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: '<your_ssh_connection_name>',
                                     transfers: [
                                         sshTransfer(cleanRemote: false,
                                                       excludes: '',
                                                       execCommand: 'docker-compose -f /path/to/docker-compose.yml up -d',
                                                       execTimeout: 120000,
                                                       flatten: false,
                                                       makeEmptyDirs: false,
                                                       noDefaultExcludes: false,
                                                       patternSeparator: '[, ]+',
                                                       remoteDirectory: '/opt/my-flask-app',
                                                       removePrefix: '',
                                                       sourceFiles: 'docker-compose.yml Dockerfile .env')
                                     ],
                                     usePromotion: false,
                                     verbose: false)
                ])
            }
        }
    }
    post {
        always {
            echo 'This will always run'
            // Clean up resources if needed
        }
        success {
            echo 'Build and Deploy Successful!'
        }
        failure {
            echo 'Build or Deploy Failed!'
        }
    }
}