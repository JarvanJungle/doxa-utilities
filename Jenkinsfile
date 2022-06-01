pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))
        office365ConnectorWebhooks([
                [
                        startNotification: false,
                        notifySuccess    : false,
                        notifyFailure    : false,
                        timeout          : 30000,
                        url              : "${env.TEAMS_WEBHOOK}"
                ]]
        )
    }

    environment {
        DEV_USERID = '750655480130'
        UAT_USERID = '556257862131'
        PROD_USERID = '750655480130' // TODO: Change to production registry
        // JAVA_HOME = "/usr/local/jdk-11.0.2" // Use java 11

        JAVA_HOME = "/usr/lib/jvm/java-11-openjdk-amd64" //  usr/lib/jvm/java-11-openjdk-amd64/bin/java

    }

    stages {
        stage('Set variables') {
            parallel {
                stage('Set variables for STAGE') {
                    when { anyOf { branch 'release/develop'; branch 'release/uat'; branch 'release/stag'; branch 'release/production'; } }
                    steps {
                        script {
                            def myRepo = checkout scm
                            def gitCommit = myRepo.GIT_COMMIT
                            def gitBranch = myRepo.GIT_BRANCH
                            def branchDelimitted = gitBranch.split('/')
                            def stageName = branchDelimitted[1].trim()
                            def shortGitCommit = "${gitCommit[0..8]}"
                            def imageTag = "${shortGitCommit}-${BUILD_NUMBER}"

                            switch (stageName) {
                                case 'develop':
                                    STAGE = "develop"
                                    CLUSTER = "${env.DEV_CLUSTER}"
                                    NAMESPACE = "development"
                                    KUBE_CREDENTIALS = 'eks_dev_secret'
                                    USERID = "${env.DEV_USERID}"
                                    break
                                case 'uat':
                                    STAGE = "uat"
                                    CLUSTER = "${env.UAT_CLUSTER}"
                                    NAMESPACE = "uat"
                                    KUBE_CREDENTIALS = 'eks_uat_secret'
                                    USERID = "${env.AMS_UAT_USERID}"
									break
                                case 'stag':
                                    STAGE = "stag"
                                    CLUSTER = "${env.DEV_CLUSTER}"
                                    NAMESPACE = "stagging"
                                    KUBE_CREDENTIALS = 'eks_dev_secret'
                                    USERID = "${env.DEV_USERID}"
                                    break
                                case 'production':
                                    NAMESPACE = "production"
                                    CLUSTER = "${env.PROD_CLUSTER}"
                                    KUBE_CREDENTIALS = 'eks_prod_secret'
                                    //REPO = "${env.DEV_REPO}" // TODO: Update for production
                                    USERID = "${env.DEV_USERID}" // TODO: Update for production
                                    break
                            }
                            DEF_KUBE_CREDENTIALS = "${KUBE_CREDENTIALS}"
                            DEF_NAMESPACE = "${NAMESPACE}"
                        }
                    }
                }
            }
        }



        stage('Deploy Configmap and Secret to Kubernetes') {
            when { anyOf { branch 'release/develop'; branch 'release/uat'; branch 'release/stag'; branch 'release/production'; } }
            steps {
                withKubeConfig([credentialsId: "${DEF_KUBE_CREDENTIALS}", serverUrl: "${CLUSTER}"]) {
                    echo "Script to deploy configmap and seret to Kubernetes"
                    echo "stage name is : ${STAGE} "
                    sh "/usr/local/bin/kubectl apply -f ./deployment/${STAGE}/connex-conf.yml --namespace ${DEF_NAMESPACE}"
                    sh "/usr/local/bin/kubectl apply -f ./deployment/${STAGE}/gateway-conf.yml --namespace ${DEF_NAMESPACE}"
                    sh "/usr/local/bin/kubectl apply -f ./deployment/${STAGE}/connex-secrets.yml --namespace ${DEF_NAMESPACE}"
                    sh "/usr/local/bin/kubectl apply -f ./deployment/${STAGE}/doxa-holdings.yml --namespace ${DEF_NAMESPACE}"
                    sh "/usr/local/bin/kubectl apply -f ./deployment/${STAGE}/aws-credentials.yml --namespace ${DEF_NAMESPACE}"
                    echo "Deployed configmap and secret to Kubernetes ${STAGE} environment and ${DEF_NAMESPACE} namespace"

                }
            }
        }


 
    }
      /*** workspace clean up*/
    post { 
        always { 
            cleanWs()
        }
    }
    
    
}