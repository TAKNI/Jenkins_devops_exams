pipeline {

    agent any

    environment {
        DOCKER_ID = "rachidtakni"

        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE  = "cast-service"

        MOVIE_TAG = "v.${BUILD_ID}.0"
        CAST_TAG  = "v.${BUILD_ID}.0"

        DOCKER_PASS_CRED = "DOCKER_HUB_PASS"
        KUBECONFIG_CRED  = "config"

        HELM_CHART   = "charts"
        RELEASE_NAME = "fastapiapp"
    }

    stages {

        stage('Build Docker Images') {
            steps {
                script {
                    sh """
                      docker build -t ${DOCKER_ID}/${MOVIE_IMAGE}:${MOVIE_TAG} ./movie-service
                      docker build -t ${DOCKER_ID}/${CAST_IMAGE}:${CAST_TAG} ./cast-service
                    """
                }
            }
        }

        stage('Push Docker Images') {
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
            }
            steps {
                script {
                    sh """
                      echo "${DOCKER_PASS}" | docker login -u ${DOCKER_ID} --password-stdin
                      docker push ${DOCKER_ID}/${MOVIE_IMAGE}:${MOVIE_TAG}
                      docker push ${DOCKER_ID}/${CAST_IMAGE}:${CAST_TAG}
                    """
                }
            }
        }

        stage('Deploy DEV') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh """
                      mkdir -p .kube
                      cat ${KUBECONFIG} > .kube/config

                      helm upgrade --install ${RELEASE_NAME}-dev ${HELM_CHART} \
                        --namespace dev \
                        -f ${HELM_CHART}/values-dev.yaml \
                        --set movieService.image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set movieService.image.tag=${MOVIE_TAG} \
                        --set castService.image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set castService.image.tag=${CAST_TAG}
                    """
                }
            }
        }

        stage('Deploy STAGING') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh """
                      mkdir -p .kube
                      cat ${KUBECONFIG} > .kube/config

                      helm upgrade --install ${RELEASE_NAME}-staging ${HELM_CHART} \
                        --namespace staging \
                        -f ${HELM_CHART}/values-staging.yaml \
                        --set movieService.image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set movieService.image.tag=${MOVIE_TAG} \
                        --set castService.image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set castService.image.tag=${CAST_TAG}
                    """
                }
            }
        }

        stage('Deploy PROD') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: "Déployer en production ?", ok: "Oui"
                }
                script {
                    sh """
                      mkdir -p .kube
                      cat ${KUBECONFIG} > .kube/config

                      helm upgrade --install ${RELEASE_NAME}-prod ${HELM_CHART} \
                        --namespace prod \
                        -f ${HELM_CHART}/values-prod.yaml \
                        --set movieService.image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set movieService.image.tag=${MOVIE_TAG} \
                        --set castService.image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set castService.image.tag=${CAST_TAG}
                    """
                }
            }
        }
    }

    post {
        failure {
            mail to: "takni.rachid@gmail.com",
                 subject: "Pipeline FAILED: ${env.JOB_NAME} #${env.BUILD_ID}",
                 body: "Consulte la console Jenkins : ${env.BUILD_URL}"
        }
    }
}
