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
      sh 'wget -P ~ "https://raw.githubusercontent.com/JorisJamers/dockertest/master/hello-web_deploy.yaml"'
      sh 'mkdir -p /var/lib/jenkins/hello-node/templates/${BUILD_NUMBER}'
      sh 'mv hello-web_deploy.yaml /var/lib/jenkins/hello-node/templates/${BUILD_NUMBER}'
      sh 'sed -i -e \"s/buildNumber/\${BUILD_NUMBER}/g\" /var/lib/jenkins/hello-node/templates/${BUILD_NUMBER}/hello-web_deploy.yaml'
    }

    stage ('Deploy to the test environment'){
      sh 'kubectl --namespace=tst apply -f "/var/lib/jenkins/hello-node/templates/${BUILD_NUMBER}/hello-web_deploy.yaml"'
      test_ip = sh(returnStdout: true, script: 'kubectl --namespace=tst get services | grep -v NAME | cut -d\' \' -f10').trim()
      echo test_ip
      sh 'gcloud dns record-sets transaction remove -z=examplezonename --name="tst.joris.gluo.io"'

      slackSend (color: '#00FF00', message: "Succefully deployed to the test environment")
    }

    stage ('Deploy to the uat environment'){
      sh 'kubectl --namespace=uat apply -f "/var/lib/jenkins/hello-node/templates/${BUILD_NUMBER}/hello-web_deploy.yaml"'
      slackSend (color: '#00FF00', message: "Succefully deployed to the uat environment")
    }

    stage ('Deploy approval to production'){
      slackSend (color: '#00FF00', message: "I am waiting for your approval to deploy the build to production.")
      input "Deploy to prod?"
    }

    stage ('Deploy to the production environment'){
      sh 'kubectl --namespace=prd apply -f "/var/lib/jenkins/hello-node/templates/${BUILD_NUMBER}/hello-web_deploy.yaml"'
      slackSend (color: '#00FF00', message: "Succefully deployed to the production environment")
    }

    slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
}
