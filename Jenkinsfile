
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "warmcoots/department-app"
        APP_NAME = "department-app"
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/dxvya-gh/Q3_Dep_Portal.git'
            }
        }

        stage('Test Application') {
            steps {
                sh '''
                    set -e
                    test -s index.html
                    grep -q "Computer Science Department" index.html
                    grep -q "Courses Offered" index.html
                    grep -q "Faculty Directory" index.html
                    grep -q "Department Announcements" index.html
                    grep -q "Contact the Department" index.html
                    echo "PASS: Department portal tests completed."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push "$DOCKER_IMAGE:$IMAGE_TAG"
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'kuberconfig',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        set -e
                        kubectl --kubeconfig="$KUBECONFIG" set image \
                          deployment/$APP_NAME \
                          $APP_NAME=$DOCKER_IMAGE:$IMAGE_TAG

                        kubectl --kubeconfig="$KUBECONFIG" rollout status \
                          deployment/$APP_NAME --timeout=120s
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Q3 Department Portal pipeline finished."
        }
    }
}
