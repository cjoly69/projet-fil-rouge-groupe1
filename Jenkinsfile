pipeline {
    environment {
		ID_DOCKER = "192.168.100.10:5000"
        IMAGE_NAME = 'projet-fil-rouge-groupe1'
        IMAGE_TAG = 'v1'
        CONTAINER_NAME = 'fil-rouge-groupe1'
    }
    agent none
     /*Build image*/
    stages {
        stage('Build staging image') {
            agent any
            steps {
                script {
                    sh 'docker build --no-cache -t ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_staging .'
                }
            }
        }
        /*push in dockerhub*/
        stage('Push staging Image on local docker repository') {
            agent any
            steps {
                script {
                    sh '''
            docker push ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_staging
        '''
                }
            }
        }

        /*deploy minikube_staging*/
        stage('Deploy on K8S in staging') {
            agent any
            steps {
                script {
                    sh '''
					sed 's/___PLATFORMTAG___/_staging/g' eazytraining-deployment.yml > eazytraining-deployment-staging.yml
					scp eazytraining-deployment-staging.yml jenkins@staging:.
					ssh jenkins@staging \
					"if docker images | grep ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_staging; then docker image rm -f ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_staging; fi"
                    ssh jenkins@staging \
                    "kubectl apply -f eazytraining-deployment-staging.yml"
					while [ `ssh jenkins@staging "kubectl get pod -n eazytraining | grep eazytraining | tail -1 | cut -d' ' -f9"` != 'Running' ]; do sleep 5; done
                 '''
                }
            }
        }
         /*Unit test with Jest on staging*/
        stage('Unit test with Jest on staging') {
            agent any
            steps {
                script {
                    sh '''
					CONTAINER=`ssh jenkins@staging "kubectl get pod -n eazytraining | grep eazytraining | tail -1 | cut -d' ' -f1"`
					ssh jenkins@staging \
					"kubectl exec $CONTAINER -n eazytraining -- bash -c 'cd /var/local/node/projet-fil-rouge-groupe1 && npm test'"
         '''
                }
            }
        }
        /*Fontional test on staging*/
        stage('Fonctional test on staging') {
            agent any
            steps {
                script {
                    sh '''
                    curl http://staging:31000 | grep -i "contact@eazytraining.fr"
                '''
                }
            }
        }
		/*push in dockerhub*/
        stage('Push production Image on local docker repository') {
            agent any
            steps {
                script {
                    sh '''
            docker tag ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}_staging ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_prod
			docker push ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_prod
        '''
                }
            }
        }
        /*clean*/
        stage('Clean Staging deployment') {
            agent any
            steps {
                script {
                    sh '''
                 ssh jenkins@staging \
                 "kubectl delete -f eazytraining-deployment-staging.yml"
               '''
                }
            }
        }
        /*deploy prod*/
        stage('Deploy on K8S in production') {
            agent any
            steps {
                script {
                    sh '''
					sed 's/___PLATFORMTAG___/_prod/g' eazytraining-deployment.yml > eazytraining-deployment-production.yml
					scp eazytraining-deployment-production.yml jenkins@production:.
					ssh jenkins@production \
					"if docker images | grep ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_prod; then docker image rm -f ${ID_DOCKER}/$IMAGE_NAME:${IMAGE_TAG}_prod; fi"
                    ssh jenkins@production \
                    "kubectl apply -f eazytraining-deployment-production.yml"
                 '''
                }
            }
        }
    }
}
