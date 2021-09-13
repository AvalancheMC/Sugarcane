pipeline {
    agent { label 'master' }
    options { timestamps() }
    stages {
        stage('Cleanup') {
            tools {
                jdk "JDK 16"
            }
            steps {
                // scmSkip(deleteBuild: true, skipPattern:'.*\\[ci skip\\].*') // This breaks building
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
                    git config user.email "jenkins@avalanchemc.org"
                    git config user.name "Jenkins"
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
    }
}
