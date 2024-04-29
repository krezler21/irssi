pipeline {
    agent any

    stages {
        stage('Pull') {
            steps {
                echo "Pullowanie repo"
                git branch: 'master', credentialsId: 'bfd3d51b-874c-4e9b-b867-26d457d62113', url: 'https://github.com/krezler21/irssi'
            }
        }
        
        stage('Build') {
            steps {
                echo "Budowa projektu"
                sh '''
                    docker build -t irssi-build:latest -f ./budowa/Dockerfile .

                    docker run --name build_container irssi-build:latest
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo "Testowanie projektu"
                sh '''
                    docker build -t irssi-test:latest -f ./test/Dockerfile .

                    docker run --name test_container irssi-test:latest
                '''
            }
        }

    }

     post{
        always{
            echo "Archiving artifacts"

            archiveArtifacts artifacts: 'artifact_*.tar.gz', fingerprint: true
            sh '''
            chmod +x cleanup.sh
            ./cleanup.sh
            '''
        }

     }
}