pipeline {

    agent any

    environment {
        MYSQL_ROOT_LOGIN = credentials('my-sql-account')
        IMAGE_NAME = 'nodejs'
        TIMESTAMP = sh(returnStdout: true, script: "date +%d.%m.%Y").trim()
        TAG = "${TIMESTAMP}-v${BUILD_NUMBER}"
    }
    stages {
        stage ('Define TAG') {
            steps {
                script {
                	echo "TAG: $TAG"
                    writeFile file: 'tag.txt', text: "$TAG"
                }
            }
        }
        stage('Packaging/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t donhatanh2000/nodejs:$TAG .'
                    sh 'docker tag donhatanh2000/nodejs:${TAG} donhatanh2000/nodejs:latest'
                    sh 'docker push donhatanh2000/nodejs:${TAG}'
                    sh 'docker push donhatanh2000/nodejs:latest'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop nhatanh-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm nhatanh-mysql-data || echo "no volume"'
                sh "docker run --name nhatanh-mysql -p 3306:3306 --network dev -v nhatanh-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d --restart always mysql "
                sh 'sleep 20'
                sh "docker exec -i nhatanh-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy nodejs to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull donhatanh2000/nodejs'
                sh 'docker container stop nhatanh-nodejs || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '
                sh 'docker container run -d --name nhatanh-nodejs -p 3000:3000 --network dev --restart always donhatanh2000/nodejs'
            }
        }
        
         stage('Deploy nginx to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull nginx'
                sh 'docker container stop nhatanh-nginx || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '
                sh 'docker container run -d --name nhatanh-nginx -p 80:80 --network dev --restart always nginx'
                sh 'docker cp ./nginx.conf nhatanh-nginx:/etc/nginx/nginx.conf'
                sh "docker restart nhatanh-nginx"
            }
        }
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
