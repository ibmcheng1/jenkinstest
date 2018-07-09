def volumes = [ hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock') ]
//volumes += secretVolume(secretName: 'microclimate-registry-secret', mountPath: '/jenkins_docker_sec')
volumes += secretVolume(secretName: 'sa-default', mountPath: '/jenkins_docker_sec')

podTemplate(label: 'icp-liberty-build',
            nodeSelector: 'beta.kubernetes.io/arch=amd64',
    containers: [
        containerTemplate(name: 'maven', image: 'maven:3.5.3-jdk-8', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker', image: 'docker:17.12', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'kubectl', image: 'ibmcom/k8s-kubectl:v1.8.3', ttyEnabled: true, command: 'cat'),
    ],
    volumes: volumes
)

{
    node ('icp-liberty-build') {
        def gitCommit
        def imageTag
        def IMAGE_TAG
        def TAG_NEW_String
        
        def OFFSET_DIR="chart/jenkinstest"		
		def TARGET_FILE="values.yaml"
		def BUILD_PROPERTIES_FILE="build.properties"
		def TAG_OLD_String="@@@TAG@@@"
      
        def UCD_APP_NAME = "Jenkinstest"
        def UCD_COMPONENT_NAME = "JenkinsTest"
        def UCD_COMPONENT_TEMPLATE = "HelmChartTemplate"
        def UCD_DELIVERY_BASE_DIR = null
        def UCD_DELIVERY_PUSH_VERSION = null       
        def UCD_Deploy_Env = "Dev"
        def UCD_Deploy_Process = "Deploy"
        def UCD_Deploy_Version = null              
        
        
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

        stage ('Push to UCD...') {
            imageTag = gitCommit
            IMAGE_TAG = gitCommit
            TAG_NEW_String=gitCommit
            
            UCD_DELIVERY_BASE_DIR = WORKSPACE + "/" + OFFSET_DIR
			UCD_DELIVERY_PUSH_VERSION = BRANCH_NAME + "." + BUILD_NUMBER
			UCD_Deploy_Version = UCD_COMPONENT_NAME + ":" + BRANCH_NAME + "." + BUILD_NUMBER
            TARGET_FILE =  UCD_DELIVERY_BASE_DIR + "/" + TARGET_FILE
            
            sh """
            #!/bin/bash
	        echo "Current Directory:"
	    	pwd
	    	echo "Current Directory File Structure:"
	    	ls -l
			echo "-------------------------"
			echo "Environment Information: "
			echo "-------------------------"
			echo "gitCommit = ${gitCommit}"
	        echo "imageTag = ${imageTag}" 
	        echo "IMAGE_TAG = ${IMAGE_TAG}"
	        echo "WORKSPACE = ${WORKSPACE}"
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
        
 	    	step([$class: 'UCDeployPublisher',
	            siteName: 'UCD-Server',
	            component: [
	                $class: 'com.urbancode.jenkins.plugins.ucdeploy.VersionHelper$VersionBlock',
	                componentName: "${UCD_COMPONENT_NAME}",
	                createComponent: [
	                    $class: 'com.urbancode.jenkins.plugins.ucdeploy.ComponentHelper$CreateComponentBlock',
	                    componentTemplate: "${UCD_COMPONENT_TEMPLATE}",
	               	    componentApplication: "${UCD_APP_NAME}"
	            	],
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
