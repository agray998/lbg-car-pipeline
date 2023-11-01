pipeline {
    agent any
    environment {
        SERVER_URL = "http://35.195.123.173:8000/" // replace with IP of second server
        MYSQL_ROOT_PASSWORD = credentials('MYSQL_ROOT_PASSWORD')
    }
    stages {
        stage('Checkout source repos') {
            steps {
                dir("lbg-car-front") {
                    git url: "https://github.com/agray998/lbg-car-react-starter", branch: "main"
                }
                dir("lbg-car-back") {
                    git url: "https://github.com/agray998/lbg-car-spring-app-starter", branch: "main"
                }
            }
        }
        stage('Test and build spring backend') {
            steps {
                dir("lbg-car-back") {
                    // sh "mvn clean test"
                    sh '''
                    cat - > src/main/resources/application.properties <<EOF
                    spring.profiles.active=prod
                    logging.level.root=DEBUG
                    server.port=8000
                    spring.jpa.show-sql=true
                    '''
                    sh "docker build -t agray998/lbg-car-back:v${BUILD_NUMBER} --build-arg MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} ."
                    sh "docker tag agray998/lbg-car-back:v${BUILD_NUMBER} agray998/lbg-car-back:latest"
                }
            }
        }
        stage('Test and build react frontend') {
            steps {
                dir("lbg-car-front") {
                    sh """
                    npm install
                    yarn test
                    docker build --build-arg SERVER_URL=${SERVER_URL} -t agray998/lbg-car-front:v${BUILD_NUMBER} .
                    docker tag agray998/lbg-car-front:v${BUILD_NUMBER} agray998/lbg-car-front:latest
                    """
                }
            }
        }
        stage('Push docker images and cleanup') {
            steps {
                sh "docker push agray998/lbg-car-front:v${BUILD_NUMBER}"
                sh "docker push agray998/lbg-car-front:latest"
                sh "docker push agray998/lbg-car-back:v${BUILD_NUMBER}"
                sh "docker push agray998/lbg-car-back:latest"
                sh "docker rmi agray998/lbg-car-back agray998/lbg-car-back:v${BUILD_NUMBER} agray998/lbg-car-front agray998/lbg-car-front:v${BUILD_NUMBER}"
            }
        }
        stage('Deploy to server') {
            steps {
                sh "scp -i ~/.ssh/server_key docker-compose.yaml jenkins@adam-cardb-server:/home/jenkins/docker-compose.yaml"
                sh """
                ssh -i ~/.ssh/server_key jenkins@adam-cardb-server <<EOF
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
