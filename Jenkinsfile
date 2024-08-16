def COLOR_MAP = ['SUCCESS': 'good', 'FAILURE': 'danger', 'UNSTABLE': 'danger', 'ABORTED': 'danger']

pipeline {
    agent { label 'docker-node-label' }

    tools {
        maven 'Maven'
    }

    environment {
        ArtifactId = readMavenPom().getArtifactId()
        Version = readMavenPom().getVersion()
        GroupId = readMavenPom().getGroupId()
        Name = readMavenPom().getName()

        NameFolder = "${env.BUILD_ID}" + '.' + "${ env.GIT_COMMIT[0..6]}"
    }

    stages {
        stage('Build') {
            steps {
                echo 'Build'
                sh "sed -i 's|<version>0.0.1</version>|<version>${env.NameFolder}</version>|g' pom.xml"
                sh 'mvn clean install package'
            }
        }

        stage('Publish to Nexus') {
            steps {
                echo 'Publish to Nexus'
                script {
                    def NexusRepo = Version.endsWith('SNAPSHOT') ? 'MyLab-SNAPSHOT' : 'MyLab-RELEASE'
                    nexusArtifactUploader artifacts:
                    [
                        [
                            artifactId: "${ArtifactId}",
                            classifier: '',
                            file: "target/${ArtifactId}-${env.NameFolder}.war",
                            type: 'war'
                        ]
                    ],
                    credentialsId: 'NEXUS',
                    groupId: "${GroupId}",
                    nexusUrl: '172.23.10.53:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: "${NexusRepo}",
                    version: "${env.NameFolder}"
                }
            }
        }

        stage('Deploy compose') {
            steps {
                sh '''
                curl -u admin:password -L "http://172.23.10.53:8081/service/rest/v1/search/assets/download?sort=version&repository=MyLab-RELEASE&maven.groupId=com.mylab&maven.artifactId=MyLab&maven.extension=war" -H "accept: application/json" --output latest.war
                docker compose down
                docker compose --project-name ${ArtifactId} build
                docker compose up -d
                '''
            }
        }

        /*stage('Deploy to Docker') {
            steps {
                echo 'Deploy to Docker'
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'Ansible',
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    execCommand: 'cd playbooks/ && ansible-playbook playbook.yml -i inventory.txt',
                                    execTimeout: 120000,
                                    flatten: false,
                                    makeEmptyDirs: false,
                                    noDefaultExcludes: false,
                                    patternSeparator: '[, ]+',
                                    remoteDirectory: '/playbooks',
                                    remoteDirectorySDF: false,
                                    removePrefix: '',
                                    sourceFiles: 'playbook.yml, inventory.txt'
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: false
                        )
                    ]
                )
            }
        }*/
    }

    /*post {
        always {
            script {
                #def commit = sh(returnStdout: true, script: 'git log --format="%H%n%an%n%s" -n 1').trim().split('\n')
                #slackSend color: COLOR_MAP[currentBuild.currentResult], message: "*Build and deploy successful* :white_check_mark:\n\nJob: `${env.JOB_NAME}`\nBuild Number: *`<${env.BUILD_URL} |${env.BUILD_NUMBER}>`*\nCommit: `${commit[2]}`\nAuthor: `${commit[1]}`\nCommit ID: *`<https://github.com/mhviet2001/CI-CD-Pipeline-Java-WebApp/commit/${commit[1]} |${commit[0][0..6]}>`*", channel: '#general'
                sh 'echo "sukses"'
            }
        }
    }*/
}
