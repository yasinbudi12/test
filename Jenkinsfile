pipeline {
    agent { label 'master' }
    environment {
        registry = "docker.io/yasinbudi12/tes"
        serviceName= 'tes'
        dockerImage = ''
        branchname = "staging"
    }
    stages {
        stage("Clone Code") {
            steps {
                script {
                checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: 'https://yasinbudi12@bitbucket.com/yasinbudi12/tes.git',
                credentialsId: 'credential-git']], branches: [[name: "${branchname}" ]]],poll: false
                env.HASH = sh(script: "echo \$(git rev-parse --short HEAD)",returnStdout: true).trim()
                env.VERSION = "${env.BUILD_NUMBER}-${branchname}-${env.HASH}"
                }
            }
        }
        stage('Building Image') {
            steps{
                slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Build - (${env.BUILD_URL})")
                bitbucketStatusNotify(buildState: 'INPROGRESS')
                script {
                  try {
                    dockerImage = docker.build("${registry}:$env.VERSION")
                    bitbucketStatusNotify(buildState: 'SUCCESSFUL')
                    }
                    catch (Exception e) {
                    bitbucketStatusNotify(buildState: 'FAILED')
                    }
                }
            }
            post {
                success {
                slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Build - (${env.BUILD_URL})")
                }   
                failure {
                slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Build - (${env.BUILD_URL})")
                }
            }
        }
        stage('Push Image') {
            steps{ 
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com', 'credential-docker' ) {
                    dockerImage.push("$env.VERSION")
                    }
                }
            }
        }
        stage('Remove Unused Docker Image') {
            steps{
                script {
                    sh "docker rmi $registry:$env.VERSION"
                }
            }
        }
        stage('Deploy to Docker') {
            steps{
                slackSend (color: '#FFFF00', message: "STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Deploy to Staging - (${env.BUILD_URL})")
                bitbucketStatusNotify(buildState: 'INPROGRESS')
                script {
                    try {
                        sh "sed -i 's/tes:latest/tes:$env.VERSION/g' docker-compose.yml"
                        //for deploy to this host
                        sh "docker-compose up -d"
                        //for deploy to other host, need to ssh like this sh "ssh -v {name_host}@{ip_host} -p {port_ssh} 'docker-compose up -d'"
                        bitbucketStatusNotify(buildState: 'SUCCESSFUL')
                    }
                    catch (Exception e) {
                    bitbucketStatusNotify(buildState: 'FAILED')
                    }
                }
            }
            post {
                success {
                slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Deploy to Staging - (${env.BUILD_URL})")
                }      
                failure {
                slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' - Deploy to Staging - (${env.BUILD_URL})")
                }
            }
        }
    }
    post {
        success {
        slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Pipeline Success (${env.BUILD_URL})")
        }
        failure {
        slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Pipeline Failed (${env.BUILD_URL})")
        }
    }  
}
