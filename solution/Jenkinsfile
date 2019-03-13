def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(
    label: label, 
    containers: [
        containerTemplate(name: 'node', image: 'node:carbon-jessie', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
    ],
    serviceAccount: 'jenkins-teamX',
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
    ]
) {
  node(label) {
    stage('Checkout repo') {
        checkout scm
    }

    stage('Test NPM') {
        container('node') {
            sh('npm install')
            sh('npm test')
        }
    }

    stage('Create Docker images') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockerhub',
          usernameVariable: 'DOCKER_HUB_USER',
          passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
          sh """
            docker login -u ${DOCKER_HUB_USER} -p ${DOCKER_HUB_PASSWORD}
            docker build -t <dockerhub-user>/<image-name>:<image-tag> .
            docker push <dockerhub-user>/<image-name>:<image-tag>
            """
        }
      }
    }

    stage('Deploy') {
        container('kubectl') {
	        sh('kubectl run --image=<dockerhub-user>/<image-name>:<image-tag> date-display-app')
        }
    }
  }
}
