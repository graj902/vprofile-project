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
        NEXUSIP = '172.31.42.72' // This is your Nexus Private IP
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vprofile--maven-group' // Correct double-dash name
        NEXUS_LOGIN = 'nexuslogin'
    }

    stages {
        stage('Build'){
            steps {
                // We use 'package' to create the .war file
                // We also skip tests here because we have a dedicated Test stage
                sh 'mvn -s settings.xml -DskipTests package'
            }
        } 
        
        stage('Test'){
            steps {
                // This command runs the tests and generates the JaCoCo report
                sh 'mvn -s settings.xml test'
            }
        } 
        
        stage ('Checkstyle') {
            steps {
                // This runs the code style analysis
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
    } // <-- The 'stages' block ENDS HERE

    //
    // This is the correct 'post' block that runs AFTER all stages
    //
    post {
        success {
            steps {
                echo 'Build was successful. Archiving the .war file...'
                // This command finds the .war file in the target directory
                // and saves it as a build artifact.
                archiveArtifacts artifacts: 'target/vprofile-v2.war', fingerprint: true
            }
        }
        
        failure {
            steps {
                echo 'Build failed!'
                // In a real project, you would add a Slack or email notification here
            }
        }
    }
}