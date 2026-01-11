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
        ansiColor()
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
    } 
}