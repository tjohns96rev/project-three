pipeline{ // the entire Jenkins Job needs to go inside the pipeline section

    agent{
        //this is where we tell Jenkins what agent to use for this build (reference to jenkins value file)
        kubernetes{
            // this tells Jenkins to use the pod called "devops" we defined in the jenkins-values.yaml file
            // which will give us access to the docker commands we need to build/push our docker image
            inheritFrom "devops"
        }
    }

    environment{
        // any environment variables we want to use can go in here
        // I recommand setting variables for the docker registry (which doubles as the image name)
        // and a variable to represent the image itself
        DEVOPS_REGISTRY='tjohns96rev/project-three'
        DEVOPS_IMAGE=''
    }
    node{
        stage ("Re-Deploy Pod Image"){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', 
                    credentialsId: 'docker-creds', namespace: '', restrictKubeConfigAccess: false, 
                    serverUrl: 'http://a4254fba4b9964a3e959069b36824855-240731871.us-east-1.elb.amazonaws.com/') {
                        sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"'  
                        sh 'chmod u+x ./kubectl'  
                        sh './kubectl apply -f blue-planetarium-deployment.yml'
                    }
                }
            }
        }
    }
    stages{
        // this is where the steps of the job will be defined

        stage("Build and push docker image"){
            // steps is where the actual commands go
            steps{
                echo "Print something to console"
                container('docker'){
                        script{
                        // the script section is sometimes needed when using functions provided by Jenkins plugins
                        //
                        DEVOPS_IMAGE= docker.build(DEVOPS_REGISTRY,".") 
                        // if docker file is store in sre file then second prameter should bd "src"
                        // withRegistry(repo location empty string if docker hub, docker credentials)
                        docker.withRegistry("", 'docker-creds'){
                            DEVOPS_IMAGE.push("latest")
                            DEVOPS_IMAGE.push("$BUILD_NUMBER")
                        } //docker-creds is the credentical id when you created yours not the same one
                    }
                }
            }
        }
        // stage ("Deploy to Kubernetes"){
        //     steps {

        //         echo "Deploy to k8s"
        //             script {
        //                 kubernetesDeploy(configs: /*"blue-planetarium-deployment.yml"*/, kubeconfigId: /*"kubeconfig"*/, enableConfigSubstitution: true) 
        //             }
        //         }
        // }
    }
}