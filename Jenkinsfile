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
            sh "mvn sonar:sonar -Dsonar.host.url=http://51.124.5.78/sonarqube -Dsonar.login=2452125946f9760df1c1214bbcecdaba223b0bcc"
        } catch(error){
            echo "The sonar server could not be reached ${error}"
        }
     }

    stage("Image Prune"){
        	imagePrune(CONTAINER_NAME)
    }

    stage('Image Build'){
		sh("docker build -t ${app_name}:${app_tag} --pull --no-cache .")
		echo "Image build complete"
    }

    stage('Push to Docker Registry'){
        	withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            		pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
        	}
    }

    stage('Deploy Application on K8s') {
    	container('kubectl'){
		withKubeConfig([credentialsId: 'dockerHubAccount',
		serverUrl: env.K8s_SERVER_URL,
		contextName: env.K8s_CONTEXT_NAME,
		clusterName: env.K8s_CLUSTER_NAME]){
			sh("kubectl apply -f ${app_name}.yaml -n debjyoti")
		}     
    		echo "Application started on port: HTTP_PORT (http)"
	}
    }

}

def imagePrune(containerName){
    try {
        sh "docker image prune -f"
        sh "docker stop $containerName"
    } catch(error){}
}


def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "docker login -u $dockerUser -p $dockerPassword"
    sh "docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

