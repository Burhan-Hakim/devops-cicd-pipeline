pipeline {
    agent none

    environment {
        IMAGE = "hakimburhan/abode-website"
        TAG   = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Job1 - Build') {
            agent { label 'built-in' }
            steps {
                checkout scm
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        docker build -t $IMAGE:$TAG .
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push $IMAGE:$TAG
                    '''
                }
            }
        }

        stage('Job2 - Test') {
            agent { label 'test' }
            steps {
                sh '''
                    docker rm -f abode-test || true
                    docker pull $IMAGE:$TAG
                    docker run -d --name abode-test -p 80:80 $IMAGE:$TAG
                    sleep 10
                    curl -f http://localhost/
                    docker exec abode-test ls /var/www/html
                '''
            }
        }

        stage('Job3 - Prod') {
            agent { label 'prod' }
            when {
                beforeAgent true
                branch 'master'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker pull $IMAGE:$TAG
                        docker tag  $IMAGE:$TAG $IMAGE:latest
                        docker push $IMAGE:latest
                        docker rm -f abode-prod || true
                        docker run -d --name abode-prod -p 80:80 --restart always $IMAGE:latest
                    '''
                }
            }
        }
    }
}
