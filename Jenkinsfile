def gitCommit
def volumes = [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock') ]
volumes += secretVolume(secretName: 'jenkins-docker-sec', mountPath: '/jenkins_docker_sec')
podTemplate(label: 'icp-liberty-build-jenkinstest', slaveConnectTimeout: 600,
    containers: [
        containerTemplate(name: 'jnlp', image: 'mycluster.icp:8500/default/jenkins/jnlp-slave:3.23-1'),
        containerTemplate(name: 'maven', image: 'mycluster.icp:8500/default/maven:3.5.4-jdk-8', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'mycluster.icp:8500/default/docker:17.12', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'mycluster.icp:8500/default/ibmcom/k8s-kubectl:v1.8.3', ttyEnabled: true, command: 'cat'),       
    ],
    volumes: volumes
)
{
    node ('icp-liberty-build-jenkinstest') {
         //def gitCommit
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
        
        stage ('ucd') {
		  def componentName = "JenkinsTest"
		  def applicationName = "JenkinsTestApp"
		  def deploymentEnvironment = "DEV"
		  def deployProcess = "Deploy"
		  def pushArtifact = true
		  def imageTag = ""
		  def IMAGE_TAG
		  def TAG_NEW_String
		
		  def OFFSET_DIR="chart/" + componentName
		  def TARGET_FILE="values.yaml"
		  def BUILD_PROPERTIES_FILE="build.properties"
		  def TAG_OLD_String="@@@TAG@@@"
		
		  def UCD_APP_NAME = applicationName
		  def UCD_COMPONENT_NAME = componentName
		  def UCD_COMPONENT_TEMPLATE = "HelmChartTemplate"
		  def UCD_DELIVERY_BASE_DIR = null
		  def UCD_DELIVERY_PUSH_VERSION = null
		  def UCD_Deploy_Env = deploymentEnvironment
		  def UCD_Deploy_Process = deployProcess
		  def UCD_Deploy_Version = null
		
		  imageTag = gitCommit
		  IMAGE_TAG = gitCommit
		  TAG_NEW_String=gitCommit
		
		  UCD_DELIVERY_BASE_DIR = WORKSPACE + "/" + OFFSET_DIR
		  UCD_DELIVERY_PUSH_VERSION = BRANCH_NAME + "." + BUILD_NUMBER
		  UCD_Deploy_Version = UCD_COMPONENT_NAME + ":" + BRANCH_NAME + "." + BUILD_NUMBER
		  TARGET_FILE =  UCD_DELIVERY_BASE_DIR + "/" + TARGET_FILE
		
		  sh """
		  #!/bin/bash
		  pwd
		  ls -l
		  echo "imageTag: ${imageTag}"
		  echo "BUILD_NUMBER: ${BUILD_NUMBER}"
		  echo "WORKSPACE: ${WORKSPACE}"
		  echo "UCD_APP_NAME = ${UCD_APP_NAME}"
		  echo "UCD_COMPONENT_NAME = ${UCD_COMPONENT_NAME}"
		  echo "UCD_Deploy_Env = ${UCD_Deploy_Env}"
		  echo "UCD_DELIVERY_BASE_DIR = ${UCD_DELIVERY_BASE_DIR}"
		  echo "UCD_DELIVERY_PUSH_VERSION = ${UCD_DELIVERY_PUSH_VERSION}"
		  echo "UCD_Deploy_Process = ${UCD_Deploy_Process}"
		  echo "UCD_Deploy_Version = ${UCD_Deploy_Version}"
		  echo "TARGET_FILE = ${TARGET_FILE}"
		  echo "-------------------------"
		  echo "Verify target file: ${TARGET_FILE}"
		  ls -l ${TARGET_FILE}
		  echo "update TAG before: "
		  cat ${TARGET_FILE} | grep tag:
		  sed -i "s|@@@TAG@@@|${imageTag}|g" ${TARGET_FILE}
		  echo "update TAG before: "
		  cat ${TARGET_FILE} | grep tag:
		  """
		
		  if (pushArtifact) {
		      step([$class: 'UCDeployPublisher',
		      siteName: 'UCD-Server',
		      component: [
		          $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
		          componentName: "${UCD_COMPONENT_NAME}",
		          delivery: [
		              $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeliveryHelper$Push',
		              pushVersion: "${UCD_DELIVERY_PUSH_VERSION}",
		              baseDir: "${UCD_DELIVERY_BASE_DIR}",
		              fileIncludePatterns: '/**',
		              fileExcludePatterns: '',
		
		              pushDescription: 'Pushed from Jenkins',
		              pushIncremental: false
		          ]
		      ]
		      ])
		  }
		
		  step([$class: 'UCDeployPublisher',
		      siteName: 'UCD-Server',
		      deploy: [
		          $class: 'com.urbancode.jenkins.plugins.ucdeploy.DeployHelper$DeployBlock',
		          deployApp: "${UCD_APP_NAME}",
		          deployEnv: "${UCD_Deploy_Env}",
		          deployProc: "${UCD_Deploy_Process}",
		          deployVersions: "${UCD_Deploy_Version}",
		          deployOnlyChanged: false
		      ]
		  ])		
		}

    }
}
