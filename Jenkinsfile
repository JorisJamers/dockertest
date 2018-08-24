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
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            /* sh 'docker pull jollygnome/hellonode:latest'
             * sh 'kubectl --namespace=prd run hello-web --image=jollygnome/hellonode --port 8000'
             * sh 'kubectl --namespace=prd expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8000'
             * sh 'kubectl --namespace=prd autoscale deployment hello-web --min=2 --max=10'*/
            EXT_IP = sh kubectl --namespace=prd get services | grep -v IP | cut -d' ' -f10
            slackSend (color: '#00FF00', message: ${EXT_IP})
        }
    }
}
