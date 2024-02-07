pipeline {
    agent any
    stages {
        stage('Initialization'){
            steps{
                echo "initialization"
                sh 'which cmake'
                sh 'which make'
            }
        }
        stage('Build') {
            steps{
                echo "Build"
            }
        }
        stage('Test'){
            steps{
                echo "Test"
            }
        }
        stage('Deploy'){
            steps{
                echo "Deploy"
            }
        }
    }
}
