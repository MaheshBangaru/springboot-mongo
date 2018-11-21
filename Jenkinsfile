node{
     
    stage('SCM Checkout'){
        git url: 'https://github.com/devopstrainingbanglore/springboot-mongo.git',branch: 'master'
    }
    
    /** stage("Gradle Clean Build"){
      def GradleHome = tool name: 'Gradle-4.10.2', type: 'gradle'
      def gradleCMD = "${GradleHome}/bin/gradle"
      sh "${gradleCMD} clean build"
      
    } **/
    
    stage('Gradle Clean Build With Wrapper'){
        //With out below permision clean build will not work
        sh 'chmod +x gradlew'
        sh './gradlew clean build'
    }
    
    stage('Build Docker Image'){
        sh 'docker build -t dockerhandson/springmongo .'
    }
    
    stage('Push Docker Image'){
        withCredentials([string(credentialsId: 'Docker_Hub_Pwd', variable: 'Docker_Hub_Pwd')]) {
          sh "docker login -u dockerhandson -p ${Docker_Hub_Pwd}"
        }
        sh 'docker push dockerhandson/springmongo'
     }
     
      stage('Run Docker Image In Dev Server'){
        
        def dockerRun = ' docker-compose up  -d'
         sshagent(['DOCKER_SERVER']) {
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.20.72'
          sh 'scp  docker-compose.yml ubuntu@172.31.20.72:/home/ubuntu'
          sh "ssh  ubuntu@172.31.20.72 docker-compose down --rmi all || true"
          sh "ssh  ubuntu@172.31.20.72 ${dockerRun}"
       }
       
