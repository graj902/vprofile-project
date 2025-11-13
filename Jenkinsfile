pipeline {
    agent any
    tools {
        maven "Maven3.9"
        jdk "JDK17"
    }
    
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Maafa143@#'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vprofile-maven-central'
        NEXUSIP = '172.31.42.72'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile--maven-group'
        NEXUS_LOGIN = 'nexuslogin'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests package'
            }
        } 
        
        stage('Test'){
            steps {
                sh 'mvn -s settings.xml test'
            }
        } 
        
        stage ('Checkstyle') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
    } // <-- The 'stages' block ENDS HERE

    //
    // THIS 'post' BLOCK IS NOW SYNTACTICALLY CORRECT
    // (I have removed the extra 'steps' wrappers)
    //
    post {
        success {
            echo 'Build was successful. Archiving the .war file...'
            // These commands are now directly inside the 'success' block
            archiveArtifacts artifacts: 'target/vprofile-v2.war', fingerprint: true
        }
        
        failure {
            echo 'Build failed!'
            // This is where you would add a Slack notification
        }
    }
}