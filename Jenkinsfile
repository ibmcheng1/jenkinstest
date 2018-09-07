def gitCommit
def volumes = [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock') ]
volumes += secretVolume(secretName: 'jenkins-docker-sec', mountPath: '/docker_reg_sec')
podTemplate(label: 'icp-liberty-build-2', slaveConnectTimeout: 600,
    containers: [
        containerTemplate(name: 'jnlp', image: 'mycluster.icp:8500/default/jenkins/jnlp-slave:3.10-1'),
        containerTemplate(name: 'maven', image: 'mycluster.icp:8500/default/maven:3.5.4-jdk-8', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/docker:17.12', ttyEnabled: true, command: 'cat'),
    ],
    volumes: volumes
)
{
    node ('icp-liberty-build') {
        def gitCommit
        stage ('Extract') {
          checkout scm
          gitCommit = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          echo "checked out git commit ${gitCommit}"
        }
        stage ('maven build') {
          container('maven') {
            sh '''
            mvn clean test install
            '''
          }
        }
         stage ('docker') {
          container('docker') {
            def imageTag = "mycluster.icp:8500/jenkinstest/jenkinstest:${gitCommit}"
            echo "imageTag ${imageTag}"
            sh """
            ln -s /jenkins_docker_sec/.dockercfg /home/jenkins/.dockercfg
            mkdir /home/jenkins/.docker
            ln -s /jenkins_docker_sec/.dockerconfigjson /home/jenkins/.docker/config.json
            docker build -t jenkinstest .
            docker tag jenkinstest $imageTag
            docker push $imageTag
            """
          }
        }
        stage ('deploy') {
          container('kubectl') {
            def imageTag = null
            imageTag = gitCommit
            sh """
            #!/bin/bash
            echo "checking if jenkinstest-deployment already exists"
            if kubectl describe deployment jenkinstest-deployment --namespace jenkinstest; then
                echo "Application already exists, update..."
                kubectl set image deployment/jenkinstest-deployment jenkinstest=mycluster.icp:8500/jenkinstest/jenkinstest:${imageTag} --namespace jenkinstest
            else
                sed -i "s/<DOCKER_IMAGE>/jenkinstest:${imageTag}/g" manifests/kube.deploy.yml
                echo "Create deployment"
                kubectl apply -f manifests/kube.deploy.yml --namespace jenkinstest
                echo "Create service"
            fi
            echo "Describe deployment"
            kubectl describe deployment jenkinstest-deployment --namespace jenkinstest
            echo "finished"
            """
          }
        }
    }
}
