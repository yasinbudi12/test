# Batumbu Test NUMBER 9
 


Create pipeline with Jenkinsfile

# Preparation

Install Plugin
```
- docker plugin
- git plugin
- docker-compose plugin
- slack plugin #fornotification
- bitbucket plugin #if you use bitbucket
```
Create credential
```
- Credential git
- Credential dockerhub

```

Choose Agent 

```
agent { label 'master' }
```

Create Environment Variabel
```
registry = "docker.io/yasinbudi12/tes"
        serviceName= 'tes'
        dockerImage = ''
        branchname = "staging"
```

# Step 1 Clone Code

Clone the code from github, bitbucket and many more source code management.
```
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
```

# Step 2 Building Docker Image

Build docker image with plugin docker.
without docker plugin, you can also use shell command like ```sh 'docker build -t {name-image}:{name-tag} .' ```

You need bitbucket plugin for notify the bitbucket build status. And need slack plugin for notify to TEAM the build image status.
```
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

```

We have bitbucket plugin for 


# Step 3 Push Image to Container Registry

You need setup credential first for container registry. If you have private registry, you can change the url.
```
        stage('Push Image') {
            steps{
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com', 'credential-docker' ) {
s                    dockerImage.push("$env.VERSION")
                    }
                }
            }
        }
```

# Step 4 Remove unused Docker Image on Local Jenkins Host

You need remove the docker image because docker image already on container registry.

```
 steps{
                script {
                    sh "docker rmi $registry:$env.VERSION"
                }
            }
        }

```



# Step 5 Deploy to Docker with Docker-Compose
You need change the image tag on docker-compose.yml with sed tools to the latest image tag already build.
And need to notify bitbucket and slack team for status deployment.

```
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
```

# Latest

Notify to slack team, the status pipeline
```
post {
        success {
        slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Pipeline Success (${env.BUILD_URL})")
        }
        failure {
        slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' Pipeline Failed (${env.BUILD_URL})")
        }
    }
}
```
