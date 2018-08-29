node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("jollygnome/hellonode")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    stage ('Change the build template'){
      sh 'wget -P ~ "https://raw.githubusercontent.com/JorisJamers/dockertest/master/hello-web-deploy.yaml"'
      String[] environmentArray = ["tst", "uat", "prd"]
      for (String item : environmentArray) {
          sh "mkdir -p /var/lib/jenkins/hello-node/templates/${item}/${BUILD_NUMBER}"
          sh "cp hello-web-deploy.yaml /var/lib/jenkins/hello-node/templates/${item}/${BUILD_NUMBER}"
          sh "sed -i -e \"s/environment/${item}/g\" /var/lib/jenkins/hello-node/templates/${item}/${BUILD_NUMBER}/hello-web-deploy.yaml"
        }
      sh 'rm -rf ~/hello-web-deploy.yaml'
    }

    stage ('Deploy to the test environment'){
      sh 'kubectl --namespace=tst apply -f "/var/lib/jenkins/hello-node/templates/tst/${BUILD_NUMBER}/hello-web-deploy.yaml"'
      slackSend (color: '#00FF00', message: "Succefully deployed to the test environment")
    }

    stage ('Deploy to the uat environment'){
      sh 'kubectl --namespace=uat apply -f "/var/lib/jenkins/hello-node/templates/uat/${BUILD_NUMBER}/hello-web-deploy.yaml"'
      slackSend (color: '#00FF00', message: "Succefully deployed to the uat environment")
    }

    stage ('Deploy approval to production'){
      slackSend (color: '#00FF00', message: "I am waiting for your approval to deploy the build to production.")
      input "Deploy to prod?"
    }

    stage ('Deploy to the production environment'){
      sh 'kubectl --namespace=prd apply -f "/var/lib/jenkins/hello-node/templates/prd/${BUILD_NUMBER}/hello-web-deploy.yaml"'
      slackSend (color: '#00FF00', message: "Succefully deployed to the production environment")
    }

    stage ('Deploy the loadbalancer'){
      sh 'kubectl apply -f "https://raw.githubusercontent.com/JorisJamers/dockertest/master/hello-web-lb-tst.yaml"'
      sh 'kubectl apply -f "https://raw.githubusercontent.com/JorisJamers/dockertest/master/hello-web-lb-uat.yaml"'
      sh 'kubectl apply -f "https://raw.githubusercontent.com/JorisJamers/dockertest/master/hello-web-lb-prd.yaml"'  
      slackSend (color: '#00FF00', message: "Check the loadbalancer")
    }

    slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
