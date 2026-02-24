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
        stage('Install cert-manager') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh """
                    mkdir -p .kube
                    cat ${KUBECONFIG} > .kube/config

                    echo "Vérification de cert-manager..."

                    if ! kubectl get ns cert-manager >/dev/null 2>&1; then
                        echo "cert-manager non installé — installation en cours..."

                        # Ajouter le repo Helm
                        helm repo add jetstack https://charts.jetstack.io
                        helm repo update

                        # Installer cert-manager + CRDs
                        helm upgrade --install cert-manager jetstack/cert-manager \
                            --namespace cert-manager \
                            --create-namespace \
                            --set installCRDs=true

                    else
                        echo "cert-manager déjà installé — skip."
                    fi
                    """
                }
            }
        }

        stage('ClusterIssuer + CoreDNS + Cleanup') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh """
                    mkdir -p .kube
                    cat ${KUBECONFIG} > .kube/config

                    echo "=============================="
                    echo " 1) Vérification du ClusterIssuer"
                    echo "=============================="

                    if ! kubectl get clusterissuer letsencrypt-prod >/dev/null 2>&1; then
                        echo "ClusterIssuer absent — installation en cours..."
                        kubectl apply -f ${HELM_CHART}/cluster-issuer.yaml
                    else
                        echo "ClusterIssuer déjà installé — skip."
                    fi


                    echo "=============================="
                    echo " 2) Redémarrage de CoreDNS"
                    echo "=============================="

                    kubectl rollout restart deployment coredns -n kube-system


                    echo "=============================="
                    echo " 3) Attente du redémarrage de CoreDNS"
                    echo "=============================="

                    kubectl rollout status deployment coredns -n kube-system --timeout=120s


                    echo "=============================="
                    echo " 4) Suppression automatique des nodes NotReady"
                    echo "=============================="

                    NOTREADY_NODES=\$(kubectl get nodes --no-headers | awk '\$2=="NotReady"{print \$1}')

                    if [ -z "\$NOTREADY_NODES" ]; then
                        echo "Aucun nœud NotReady détecté. Rien à supprimer."
                    else
                        echo "Nœuds NotReady détectés :"
                        echo "\$NOTREADY_NODES"
                        echo "Suppression en cours..."
                        echo "\$NOTREADY_NODES" | xargs -r kubectl delete node
                    fi
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
                        --create-namespace \
                        -f ${HELM_CHART}/values-dev.yaml \
                        --set movieService.image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set movieService.image.tag=${MOVIE_TAG} \
                        --set castService.image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set castService.image.tag=${CAST_TAG}
                    """
                }
            }
        }

        stage('Deploy QA') {
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                script {
                    sh """
                      mkdir -p .kube
                      cat ${KUBECONFIG} > .kube/config

                      helm upgrade --install ${RELEASE_NAME}-qa ${HELM_CHART} \
                        --namespace qa \
                        --create-namespace \
                        -f ${HELM_CHART}/values-qa.yaml \
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
                        --create-namespace \
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
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials('config')
            }
            steps {
                input message: "Déployer en production ?", ok: "Oui"
                script {
                    sh """
                    mkdir -p .kube
                    cat ${KUBECONFIG} > .kube/config

                    helm upgrade --install ${RELEASE_NAME}-prod ${HELM_CHART} \
                        --namespace prod \
                        --create-namespace \
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
