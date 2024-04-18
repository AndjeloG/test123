def sendEmail(jname, jid, testr) {
    echo (message: "${jname}, ${jid}, ${testr}")
}

//pipeline structure
pipeline {
    agent any
    parameters {
        booleanParam description: 'Izvrsiti testove', name: 'RUN_TEST'
        string description: 'Git grana', name: 'GIT_BRANCH'
        booleanParam description: 'Slati email', name: 'SEND_EMAIL'
        }
    stages {
        stage('Download') {
            steps {
                cleanWs()
                echo (message: "Ovo je Download")
                dir('pipeline') {
                    git (
                        branch: "${params.GIT_BRANCH}",
                        url: 'https://github.com/KLevon/jenkins-course'
                    )
                }
                rtDownload (
                    serverId: 'Artifactory',
                    spec: '''{
                          "files": [
                            {
                                "pattern": "generic-local/printer2.zip",
                                "target": "folder.zip"
                            }
                            ]
                        }'''
                    )
                unzip(
                    zipFile: "folder.zip",
                    dir: "pipeline"
                )
            }
        }
        stage('Build') {
            steps {
                echo (message: "Ovo je Build")
                withCredentials (
                   [usernamePassword(credentialsId: 'Test1234', passwordVariable: 'psw', usernameVariable: 'usr')] 
                ) {
                    echo (message: "Username je: $usr")
                    echo (message: "Pass je: $psw")
                }
                bat (
                    script: """
                    cd pipeline
                    Makefile.bat
                """
                )
            }
        }
        stage('Tests') {
            when {
               equals expected: true,
               actual: params.RUN_TEST
            }
            steps {
                echo (message: "Ovo je Tests")
                script {
                    env.izlaz = ""
                    def niz = ["printer", "scanner", "main"]
                    for (element in niz) {
                           env.izlaz += bat (
                                script: """
                                cd pipeline
                                Tests.bat ${element}
                                """,
                                returnStdout: true
                            ).trim()
                    }
                }
            }
        }
        stage('Publish') {
            steps {
                echo (message: "Ovo je Publish")
                script {
                    zip (
                        zipFile: "pipeline.zip",
                        archive: true,
                        dir: "pipeline"
                    )
                }
                rtUpload (
                    serverId: 'Artifactory',
                    spec: """{
                          "files": [
                            {
                                "pattern": "pipeline.zip",
                                "target": "generic-local/release/${env.BUILD_ID}/"
                            }
                            ]
                        }"""
                    )
            }
        }
    }
    post {
        always {
                script {
                    sendEmail(env.JOB_NAME, "$env.BUILD_ID", "$env.izlaz")
                    if (params.SEND_EMAIL == true)
                        {
                            echo (message: "Salje se mejl!")
                        }
                    else 
                        {
                            echo (message: "Ne salje se mejl!")
                        }
                }
        }
    }
}
