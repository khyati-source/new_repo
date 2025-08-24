@Library('my_library') _

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    echo 'Building...'
                    def config = [
                        url: 'https://github.com/khyati-source/maven_calculator_app-main.git',
                        branch: 'main',
                        credentialsId: 'my-github-token'
                    ]
                    gitCheckout(config)
                    sh '''
                        pwd
                        ls -lrt
                    '''
                }
            }
        }
    }
}
      
