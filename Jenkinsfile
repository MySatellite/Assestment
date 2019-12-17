def label = "ci-poc-adira-frontend-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'curl', image: 'centos', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'mvn', image: 'maven', command: 'cat', ttyEnabled: true)
    
],
volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
])
{
    node(label) {
        try{
            stage('Jenkins Job triggered') {
                echo 'pipeline start...'
            }
            
            stage('Get Source from SCM') {
                stage = 'Get Source from SCM'
                git branch: "master",
                    credentialsId: '3446011b-6cd5-4953-99ae-6ec186d9a2bc',
                    url: 'http://redmine.klik.digital:8080/tfs/DefaultCollection/Adira/_git/frontend-application'
            }
            
            stage('Building Apps'){
                stage = 'Building Apps'
                container('mvn'){
                    sh "mvn -v"
                    sh "mvn clean package -Dmaven.test.skip=true"
                }
            }
            
            stage('Build & Containerized Application') {
                stage = 'Build & Containerized Application'
                container('docker') {
                    sh 'docker -v'
                    sh "docker build -t registry.hub.docker.com/satellite/springboot-sample:${BUILD_NUMBER} . "
                    sh "docker login -u 'chupsamir' -p 'y3rs1n1a' registry.hub.docker.com/chupsamir"
                    sh "docker push registry.hub.docker.com/satellite/springboot-sample:${BUILD_NUMBER}"
                }
            }
            
            stage('Deploy to K8s Cluster') {
                stage = 'Deploy to K8s Cluster'
                container('kubectl') {
                    sh 'kubectl apply  -f ./yaml/deployment.*'
                }
            }
            
            stage('Creating Service..') {
                stage = 'Creating Servicer'
                container('kubectl') {
                    sh 'kubectl apply -f ./yaml/service*.*'    
                }
            }
            
            stage('Prepare network..') {
                stage = 'Prepare network'
                container('kubectl') {
                    sh 'kubectl apply -f ./yaml/ingress.*'
                }
            }
            
            stage ('Notification') {
                container ('curl'){
                    sh "curl -s -X POST https://api.telegram.org/bot866817243:AAGRegqk1alChEZhUGb9mv4LvVL83TFskjs/sendMessage -d chat_id=146266195 -d text='Deployment frontend Berhasil'"
                }
            }
        } catch (e) {
            container ('curl'){
                sh "curl -s -X POST https://api.telegram.org/bot866817243:AAGRegqk1alChEZhUGb9mv4LvVL83TFskjs/sendMessage -d chat_id=146266195 -d text='Deployment frontend error pada stage ${stage}'"
            }
        }
    }
} 