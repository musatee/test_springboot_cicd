properties([pipelineTriggers([githubPush()])])
pipeline{
    agent{
        any
    }
    environment {
        repo_creds=credentials('cred_name')
    } 
   
    stages{
        stage("Cloning Repo"){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'git@github.com:musatee/helmfile_example.git']]])
            }
        }
        /* using docker container as agent instead of installing plugins on node. NOTE: current workspace will be mounted inside container 
        & no new workspace will be created. so everyting from current WS is mounted with container . It's done by mentioning "reuseNode" to true which 
        overrides default behavior of docker container agent */
        stage("Building Artifact"){
            agent {
                docker {
                    image 'maven:3.8.7-openjdk-18' 
                    reuseNode true
                }
            } 
            steps{
               sh 'mvn clean install'
            }
            post {
                    failure {
                        slackSend channel: 'channel_name', message: "Building artifact is failed with build ID: ${env.BUILD_ID}!!"
                     }
                }
        } // end of building artifact stage
        stage("Building Image & committing to dockerHub"){
            steps{
                script{
                    docker.withRegistry("https://index.docker.io/v1/","repo_creds") {
                        def app_image = docker.build("musatee11/springboot:${env.BUILD_ID}")
                        app_image.push()
                        }
                }
                
                
            }
            post {
                    always {
                        sh 'docker rmi -f $(docker images -aq)'
                     }
                    failure {
                        slackSend channel: 'channel_name', message: "Building image & commit to dockerHub is failed with build ID: ${env.BUILD_ID}!!"
                     }
                }

        }
        stage("deploy to Cluster"){
           steps{
            sh "helmfile sync  --values ./my-helmchart/prod_values.yaml --set deployment.image.tag="${env.BUILD_ID}"" 
            //sh "helmfile sync  --values ./my-helmchart/dev_values.yaml --set deployment.image.tag="${env.BUILD_ID}"" 
            //sh "helmfile sync  --values ./my-helmchart/uat_values.yaml --set deployment.image.tag="${env.BUILD_ID}"" 

             
           }
           post {
                    failure {
                        slackSend channel: 'channel_name', message: "helmfile sync operation is failed with build ID: ${env.BUILD_ID}!!"
                     }
                } 
        }
    }
   post{
        
        always {
            cleanWs()
        }
        success {
            slackSend channel: 'channel_name', message: "Deployment to EKS is successful with build ID: ${env.BUILD_ID}!!"
        }
        failure {
           slackSend channel: 'channel_name', message: "Deployment to EKS is failed on build ID: ${env.BUILD_ID}" 
        } 
       
    } 
}

