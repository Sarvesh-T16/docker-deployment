pipeline {
    agent any
    environment {
        SERVER_URL = "http://34.77.110.20:8000/" // replace with IP of target deployment server
        MYSQL_ROOT_PASSWORD = credentials('MYSQL_ROOT_PASSWORD')
    }
    stages {
        stage('Checkout source repos') {
            steps {
                dir("lbg-car-front") {
                    git url: "https://github.com/Sarvesh-T16/lbg-car-react-starter", branch: "main"
                }
                dir("lbg-car-back") {
                    git url: "https://github.com/Sarvesh-T16/lbg-car-react-starter", branch: "main"
                }
            }
        }
        stage('Build spring backend') {
            steps {
                dir("lbg-car-back") {
                    sh '''
                    cat - > src/main/resources/application.properties <<EOF
                    spring.profiles.active=prod
                    logging.level.root=DEBUG
                    server.port=8000
                    spring.jpa.show-sql=true
                    '''
                    sh "docker build -t Sarvesh-T16/lbg-car-back:v${BUILD_NUMBER} --build-arg MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} ."
                    sh "docker tag Sarvesh-T16/lbg-car-back:v${BUILD_NUMBER} Sarvesh-T16/lbg-car-back:latest"
                }
            }
        }
        stage('Build react frontend') {
            steps {
                dir("lbg-car-front") {
                    sh """
                    npm install
                    docker build --build-arg SERVER_URL=${SERVER_URL} -t Sarvesh-T16/lbg-car-front:v${BUILD_NUMBER} .
                    docker tag Sarvesh-T16/lbg-car-front:v${BUILD_NUMBER} Sarvesh-T16/lbg-car-front:latest
                    """
                }
            }
        }
        stage('Push docker images and cleanup') {
            steps {
                sh "docker push /lbg-car-front:v${BUILD_NUMBER}"
                sh "docker push Sarvesh-T16/lbg-car-front:latest"
                sh "docker push Sarvesh-T16/lbg-car-back:v${BUILD_NUMBER}"
                sh "docker push Sarvesh-T16/lbg-car-back:latest"
                sh "docker rmi Sarvesh-T16/lbg-car-back Sarvesh-T16d/lbg-car-back:v${BUILD_NUMBER} Sarvesh-T16/lbg-car-front Sarvesh-T16/lbg-car-front:v${BUILD_NUMBER}"
            }
        }
        stage('Deploy to server') {
            steps {
                sh "scp -i ~/.ssh/server_key docker-compose.yaml jenkins@34.77.110.20:/home/jenkins/docker-compose.yaml"
                sh """
                ssh -i ~/.ssh/server_key jenkins@34.77.110.20 <<EOF
                export MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
                docker-compose up -d
                """
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
