DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}-ce-alpine"

node {
    def docker_image

    stage('Validate Environment') {
        required_env = [ 'DOCKER_IMAGE_NAMESPACE_DEV',
                         'DOCKER_IMAGE_NAMESPACE_PROD',
                         'DOCKER_IMAGE_REPOSITORY',
                         'DOCKER_REGISTRY_HOSTNAME',
                         'DOCKER_REGISTRY_URI',
                         'DOCKER_REGISTRY_CREDENTIALS_ID',
                         'GITHUB_TOKEN_CREDENTIALS_ID']


        fail = 0

        required_env.each { required ->
            if(env[required] == null) {
                fail = 1
                echo "Missing required environment variable: '${required}'" 
            }
        }

        if(fail) {
            error("Missing required environment variables")
        }
    }

    stage('Clone') {
        checkout scm
    }

    stage('Build') {
        withCredentials([string(credentialsId: env.GITHUB_TOKEN_CREDENTIALS_ID, variable: 'GITHUB_TOKEN')]) {
            docker_image = docker.build("${env.DOCKER_IMAGE_NAMESPACE_DEV}/${env.DOCKER_IMAGE_REPOSITORY}", "--build-arg GITHUB_TOKEN=${GITHUB_TOKEN} --file ce/alpine/Dockerfile ." )
        }
    }

    stage('Test') {
        /* Figure out how to get this to work on a stand alone Jenkins instance running with the -u jenkins_uid:jenkins_gid
        docker_image.inside {
            sh 'echo "Tests passed"'
        }
        */
    }

    stage('Push') {
        docker.withRegistry(env.DOCKER_REGISTRY_URI, env.DOCKER_REGISTRY_CREDENTIALS_ID) {
            docker_image.push(DOCKER_IMAGE_TAG)
        }
    }

    stage('Scan') {
        httpRequest acceptType: 'APPLICATION_JSON', authentication: env.DOCKER_REGISTRY_CREDENTIALS_ID, contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, responseHandle: 'NONE', url: "${env.DOCKER_REGISTRY_URI}/api/v0/imagescan/scan/${env.DOCKER_IMAGE_NAMESPACE_DEV}/${env.DOCKER_IMAGE_REPOSITORY}/${DOCKER_IMAGE_TAG}/linux/amd64"

        def scan_result

        def scanning = true
        while(scanning) {
            def scan_result_response = httpRequest acceptType: 'APPLICATION_JSON', authentication: env.DOCKER_REGISTRY_CREDENTIALS_ID, httpMode: 'GET', ignoreSslErrors: true, responseHandle: 'LEAVE_OPEN', url: "${env.DOCKER_REGISTRY_URI}/api/v0/imagescan/repositories/${env.DOCKER_IMAGE_NAMESPACE_DEV}/${env.DOCKER_IMAGE_REPOSITORY}/${DOCKER_IMAGE_TAG}"
            scan_result = readJSON text: scan_result_response.content

            if (scan_result.size() != 1) {
                println('Response: ' + scan_result)
                error('More than one imagescan returned, please narrow your search parameters')
            }

            scan_result = scan_result[0]

            if (!scan_result.check_completed_at.equals("0001-01-01T00:00:00Z")) {
                scanning = false
            } else {
                sleep 30 
            }

        }
        println('Response JSON: ' + scan_result)
    }

    stage('Promote') {
        httpRequest acceptType: 'APPLICATION_JSON', authentication: env.DOCKER_REGISTRY_CREDENTIALS_ID, contentType: 'APPLICATION_JSON', httpMode: 'POST', ignoreSslErrors: true, requestBody: "{\"targetRepository\": \"${env.DOCKER_IMAGE_NAMESPACE_PROD}/${env.DOCKER_IMAGE_REPOSITORY}\", \"targetTag\": \"${DOCKER_IMAGE_TAG}\"}", responseHandle: 'NONE', url: "${env.DOCKER_REGISTRY_URI}/api/v0/repositories/${env.DOCKER_IMAGE_NAMESPACE_DEV}/${env.DOCKER_IMAGE_REPOSITORY}/tags/${DOCKER_IMAGE_TAG}/promotion"
    }
}
