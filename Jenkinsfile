pipeline {
    agent any
    stages {
        stage ('SCM checkout') {
            steps {
                script{
                     git credentialsId: 'git-cred', url: 'https://github.com/gbgowtham/5.1_Kubernetes_CI-CD_Project_Deploy_using_Helm-chart.git'
                }
            }
        }
        stage ('SonarQube Code analysis'){
            steps {
                script{
                    def scannerHome = tool 'sonarscanner4';
                    withSonarQubeEnv('sonar-pro') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=rocket-nodejs"
                    }
                }
            }
        }
        stage ('Build') {
            steps {
                script{
                     sh 'npm install'
                }
            }
        }
        stage('Docker Build Images') {
            steps {
                script {
                    sh 'docker build -t gbgowtham/helm-rocket:v1 .'
                    sh 'docker images'
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                        sh "docker login -u gbgowtham -p ${dockerPassword}"
                        sh 'docker push gbgowtham/helm-rocket:v1'
                    }
                }
            }
        }
        stage('Deploy on k8s') {
            steps {
                script {
                    withKubeCredentials(kubectlCredentials: [[ credentialsId: 'kubernetes' ]]) {
                        sh 'kubectl create secret generic helm --from-file=.dockerconfigjson=/opt/docker/config.json  --type kubernetes.io/dockerconfigjson --dry-run=client -oyaml > secret.yaml'
                        sh 'kubectl apply -f secret.yaml'
                        sh 'helm package ./Helm'
                        sh 'helm install myrocket ./myrocketapp-0.1.0.tgz'
                        sh 'helm ls'
                        sh 'kubectl get pods -o wide'
                        sh 'kubectl get svc'
                    }
                }
            }
        }
    }
}
