pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps{
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: 'refs/tags/${VERSION}']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [], 
                    submoduleCfg: [], 
                    userRemoteConfigs: [[url: 'git@github.com:tqt-coder/nodejs-random-color.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t nodejs-random-color:${VERSION} .'
            }
        }
        stage('Upload image to ECR') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 909927813182.dkr.ecr.us-east-1.amazonaws.com'
                sh 'docker tag nodejs-random-color:${VERSION} 909927813182.dkr.ecr.us-east-1.amazonaws.com/nodejs-random-color:${VERSION}'
                sh 'docker push 909927813182.dkr.ecr.us-east-1.amazonaws.com/nodejs-random-color:${VERSION}'
            }
        }
    }
}
