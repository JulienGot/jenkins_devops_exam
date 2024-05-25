pipeline {
    environment {
        DOCKER_ID = "juliengot"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
        KUBECONFIG = credentials("config")
    }
    agent any

    stages {
        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                        sleep 6
                    '''
                }
            }
        }

        stage('Docker Run') {
            steps {
                script {
                    sh '''
                        docker run -d -p 8080:80 --name ${DOCKER_IMAGE}_${BUILD_ID} $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        sleep 10
                    '''
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        curl localhost:8080
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app fastapi --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app fastapi --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production?', ok: 'Yes'
                }
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp fastapi/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install app fastapi --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

