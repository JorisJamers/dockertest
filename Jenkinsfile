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

    script{
      try {
        stage ('Deploy to develop env}{
          sh 'docker pull jollygnome/hellonode:latest'
          sh 'kubectl --namespace=tst run hello-web --image=jollygnome/hellonode --port 8000'
          sh 'kubectl --namespace=tst expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8000'
          sh 'kubectl --namespace=tst autoscale deployment hello-web --min=2 --max=10'
        } catch (Exception e) {
            kubectl --namespace=tst set image deployments hello-web hello-web=jollygnome/hellonode:${env.BUILD_NUMBER}
          }
        }
    }
    /*stage ('Deploy to develop env'){
    *        sh 'docker pull jollygnome/hellonode:latest'
    *        sh 'kubectl --namespace=tst run hello-web --image=jollygnome/hellonode --port 8000'
    *        sh 'kubectl --namespace=tst expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8000'
    *        sh 'kubectl --namespace=tst autoscale deployment hello-web --min=2 --max=10'
    }*/
    stage ('Deploy tot uat env'){
            sh 'docker pull jollygnome/hellonode:latest'
            sh 'kubectl --namespace=uat run hello-web --image=jollygnome/hellonode --port 8000'
            sh 'kubectl --namespace=uat expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8000'
            sh 'kubectl --namespace=uat autoscale deployment hello-web --min=2 --max=10'
    }
    stage ('Deploy approval'){
            input "Deploy to prod?"
    }
    stage ('Deploy to production env'){
            sh 'docker pull jollygnome/hellonode:latest'
            sh 'kubectl --namespace=prd run hello-web --image=jollygnome/hellonode --port 8000'
            sh 'kubectl --namespace=prd expose deployment hello-web --type=LoadBalancer --port 80 --target-port 8000'
            sh 'kubectl --namespace=prd autoscale deployment hello-web --min=2 --max=10'
    }


    }
}
