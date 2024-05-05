pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }

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

                    docker cp build_container:/irssi/build ./artifacts
                    docker logs build_container > ./artifacts/log_build.txt
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo "Testowanie projektu"
                sh '''
                    docker build -t irssi-test:latest -f ./test/Dockerfile .

                    docker run --name test_container irssi-test:latest

                    docker logs test_container > ./artifacts/log_test.txt
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploy projektu"
                sh '''
                
                docker build -t irssi-deploy:latest -f ./deploy/Dockerfile .
                docker run -p 3000:3000 -d --rm --name deploy_container irssi-deploy:latest
                '''
            }
        }

        stage('Publish') {
            steps {
                echo "Publikacja projektu"
                sh '''
                TIMESTAMP=$(date +%Y%m%d%H%M%S)
                tar -czf artifact_$TIMESTAMP.tar.gz artifacts
                
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                NUMBER='''+ env.BUILD_NUMBER +'''
                docker tag irssi-deploy:latest krezler21/irssi_fork:latest
                docker push krezler21/irssi_fork:latest
                docker logout

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