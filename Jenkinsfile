// pipeline {
//     agent {
//         label "AGENT-1"
//     }
//     environment {
//         COURSE = "jenkins"
//         appVersion = ""
//         region = "us-east-1"
//         ACC_ID = "911893329385"
//         PROJECT = "roboshop"
//         COMPONENT = "catalogue"       
//     }
//     options {
//         timeout(time:30 , unit:"MINUTES")
//         disableConcurrentBuilds() 
//     }
//     parameters {
//         string(name: "appVersion", description: "Image version of the application")
//         choice(name: "deploy_to", choices:["dev","qa","prod"], description: "environment")
//     }
//     stages {
//         stage("Check status") {
//             steps {
//                 script {
//                     withAWS(credentials:"aws-creds", region:"us-east-1"){
//                         sh """
//                             aws eks update-kubeconfig --region $region --name '$PROJECT-${params.deploy_to}' 
//                             kubectl get nodes
//                             kubectl apply -f 01-namespace.yaml
//                             sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
//                             helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
//                         """
//                     }
//                 }
//             }        
//         }
//     } 
// }



pipeline {
    agent {
        label "AGENT-1"
    }
    environment {
        COURSE = "jenkins"
        appVersion = ""
        region = "us-east-1"
        ACC_ID = "911893329385"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"       
    }
    options {
        timeout(time:30 , unit:"MINUTES")
        disableConcurrentBuilds() 
    }
    parameters {
        string(name: "appVersion", description: "Image version of the application")
        choice(name: "deploy_to", choices:["dev","qa","prod"], description: "environment")
    }
    stages {
        stage("Check status") {
            steps {
                script {
                    withAWS(credentials:"aws-creds", region:"us-east-1"){
                        sh """
                            aws eks update-kubeconfig --region $region --name '$PROJECT-${params.deploy_to}' 
                            kubectl get nodes
                            kubectl apply -f 01-namespace.yaml
                            sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                            helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                        """
                    }
                }
            }        
        }
        stage("Check Deploy"){
            steps{
                script{
                    withAWS(credentials:"aws-creds",region:"us-east-1") {
                        // def deployment = sh(returnStdout:true, script:"kubectl rollout status deployment/catalogue --timeout=30s || echo FAILED" )
                        def deploymentStatus = sh(returnStdout:true, script:"kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo FAILED").trim()
                        if (deploymentStatus.contains("successfully rolled out")){
                            echo "deployment is success"
                        }
                        else{
                            sh """
                                helm rollback $COMPONENT -n $PROJECT
                                sleep  20
                            """
                            def rollback = sh(returnStdout:true, script:"kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo Failed").trim()
                            if (rollback.contains("successfully rolled out")){
                                error "deployment fail , succefully rolled out"
                            }
                            else{
                                error "deployment fail, rollback fail"
                            }
                        }
                    }
                }
            }
        }
    } 
}