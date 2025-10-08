pipeline{

    agent any
    environment{
        env = 'production'
    }
    stages{
        stage('Build'){
            steps{
                echo 'Building'   
            }
        }
        stage('Test'){
            steps{
                echo 'Testing'
            }
        }
        stage('Deploy'){
            steps{
                echo "Deplying to ${env}"
            }
        }
    }
}