pipeline {
    agent { label 'slave' }
    options { timestamps() }

   environment {
        discord_webhook1 = credentials('discord_webhook')
    }
    
    stages {
        stage('Cleanup') {
            tools {
                jdk "JDK 16"
            }
            steps {
                scmSkip(deleteBuild: true, skipPattern:'^\\[CI-SKIP\\]')
                sh 'git config --global gc.auto 0'
                sh 'rm -rf ./target'
            }
        }
        stage('Decompile & apply patches') {
            tools {
                jdk "JDK 16"
            }
            steps {
                    sh '''
                    git checkout ${BRANCH_NAME}
                    git reset --hard
                    git fetch 
                    git pull
                    git config user.email "jenkins@sugarcanemc.org"
                    git config user.name "Jenkins"
                    chmod +x gradlew
                    rm -rf Sugarcane-Server
                    rm -rf Sugarcane-API
                    ./gradlew printMinecraftVersionAP applyPatches
                    '''
                }
            }
        stage('Build') {
            tools {
                jdk "JDK 16"
            }
            steps {
                        sh'''
                        ./gradlew printMinecraftVersionBD build paperclipJar :Sugarcane-API:publishMavenPublicationToMavenRepository publishToMavenLocal
                        mkdir -p "./target"
                        cp -v "sugarcane-paperclip.jar" "./target/sugarcane-paperclip-b$BUILD_NUMBER.jar"
                        '''
            }
        }

        stage('Archive Jars') {
            steps {
                archiveArtifacts(artifacts: 'target/*.jar', fingerprint: true)
            }
        }
        
     
        stage('Discord Webhook') {
            steps {
                script {
                    discordSend description: "Build: " env.BUILD_NUMBER " Status: " env.BUILD_STATUS " Changes: - " env.BUILD_CHANGES " Artifacts: " env.BUILD_ARTIFACTS, footer: "Sugarcane " env.BUILD_NUMBER, link: env.BUILD_URL, result: currentBuild.currentResult, title: JOB_NAME, webhookURL: discord_webhook1
                }
            }   
        }
    }
}
