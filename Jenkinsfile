node('slave1') {

    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('Prepare Environment') {
        echo 'Initializing all the variables...'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out the code from Git repository...'
            git 'https://github.com/masun11/star-agile-insurance-project.git'
        } catch(Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
The Jenkins job ${JOB_NAME} has failed. Please check it immediately at the link below:
${BUILD_URL}''', 
            subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} Failed', 
            to: 'masunulla@gmail.com'
        }
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report',
            reportTitles: '',
            useWrapperFileDirectly: true
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image...'
        sh "${dockerCMD} build -t mausnulla/insure-me:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub...'
        withCredentials([usernamePassword(credentialsId: 'docker_login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh "${dockerCMD} login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
            sh "${dockerCMD} push ${DOCKER_USER}/insure-me:${tagName}"
        }
    }

    stage('Configure and Deploy to Test Server') {
        echo 'Running Ansible Playbook for deployment...'
        ansiblePlaybook become: true, 
            credentialsId: 'ansible-key', 
            disableHostKeyChecking: true, 
            installation: 'ansible', 
            inventory: '/etc/ansible/hosts', 
            playbook: 'ansible-playbook.yml'
    }

}
