pipeline 
{
    agent any

    stages 
    {
        
        stage ('Build docker image')
        {
            steps
            {
                dockerapp = docker.build("$user/kube-news:${env.BUILD_ID}", '-f ./src/Dockerfile ./src')
            }
        }
   
        stage ('Push docker image')
        {
            steps
            {
                dockerapp = docker.withRegistry("https://registry.hub.docker.com", 'dockerhub'){
                    dockerapp.push('latest')
                    dockerapp.push("${env.BUILD_ID}")
                }
            }
        }

        stage ('Deploy K8s')
        {
            steps{
                withKubeconfig([credentialsId: 'kubeconfig']){
                    sh 'kubectl apply -f ./Kubernetes/kube.yaml'
                }
            }
        }

        stage('Test POST Endpoint') {
            steps {
                sh "
                curl --header "Content-Type: application/json" \
                --request POST \
                --data '{"title":"test","content":"testcontent", "summary": "test route"}' \
                http://${ip}/post
                "
            }
            post {
                success {
                echo 'POST endpoint test succeeded!'
                }
                failure {
                echo 'POST endpoint test failed!'
                }
            }
        }

    }
}