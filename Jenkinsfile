def CONTAINER_NAME="app-332488"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="debjyotidas"
def HTTP_PORT="8090"


node {

        def app_name = 'app-332488'
	def app_dockerfile_name = 'Dockerfile'
	def app_container_name = 'app-332488'
	def app_tag="latest"


    stage('Initialize'){
        def dockerHome = tool 'myDocker'
        def mavenHome  = tool 'myMaven'
        env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
    }

    stage('Checkout') {
        checkout scm
    }

    stage('Build'){
        sh "mvn clean install"
    }

    stage('Sonar'){
        try {
            sh "mvn sonar:sonar -Dsonar.host.url=http://20.50.157.178/sonarqube -Dsonar.login=d7f92f5c317b66d4896891cdc5649f0bf2d3e5a2"
        } catch(error){
            echo "The sonar server could not be reached ${error}"
        }
     }


    stage('Build and Push to Docker Registry'){
        	withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
    			sh ("docker login -u ${USERNAME} -p ${PASSWORD} http://20.50.33.238:5000")
			sh ("docker build -t 20.50.33.238:5000/${app_name}:${BUILD_NUMBER} --pull --no-cache .")
    			sh ("docker push 20.50.33.238:5000/${app_name}:${BUILD_NUMBER}")
        	}
    		echo "Image push complete"
    }

    stage('Deploy Application on K8s') {
    		sh("curl -LO https://storage.googleapis.com/kubernetes-release/release/\$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl")
		sh("chmod +x ./kubectl")
		sh("./kubectl apply -f registrykey-332488.yaml -n debjyoti")
		sh("cat ./${app_name}.yaml | sed s/1.0.0/${BUILD_NUMBER}/g | ./kubectl apply -f -")
    		echo "Application started on port: HTTP_PORT (http)"
    }

}


