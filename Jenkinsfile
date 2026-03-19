def sendMail(name, ID, status, output) {
    echo(message: "Sending email notification...")
    echo(message: "Job ${name} with ${ID} is ${status}.")
    echo(message: "Test result: ${output}.")
}
def TEST_OUT = ""
pipeline {
    agent any
    parameters {
        string (
            name: 'ARTIFACT_NAME',
            defaultValue: 'pipeline_default.zip'
            )
        booleanParam (
            name: 'FAIL_PIPELINE',
            defaultValue: false
            )
        booleanParam (
            name: 'RUN_TEST',
            defaultValue: true
            )
        string (
            name: 'GIT_BRANCH',
            defaultValue: 'pipeline'
            )
    }
    stages {
        stage('Download') {
            steps {
                echo(message: "Download...")
                cleanWs()
                withCredentials (
                    [usernamePassword(credentialsId: 'Credentials_1', passwordVariable: 'psw', usernameVariable: 'usr')]
                    ) {
                        echo(message: "USER: ${usr}")
                        echo(message: "PASSWORD: ${psw}")
                    }
                dir ('pipeline') {
                    git (
                    branch: "${params.GIT_BRANCH}",
                    url: 'https://github.com/KLevon/jenkins-course')
                }
                rtDownload(
                    serverId: 'Artifactory_1',
                    spec: '''{
                        "files": [
                        {
                        "pattern": "generic-local/libraries/printer.zip",
                        "target": "printer.zip",
                        "flat" : "true"
                            
                        }
                        ]
                        
                    }'''
            )
            unzip (
                zipFile:"printer.zip",
                dir: "pipeline"
                )
            }
        }
        
        stage('Build') {
            steps {
                echo(message: "Build...")
                bat(
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
                echo(message: "Testing...")
                script {
                   def array = ["printer", "scanner", "main"]
                   for (element in array) {
                       TEST_OUT += bat(
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
        stage('Dynamic') {
            when {
                branch 'feature/test*'
            }
            steps {
                echo(message: "Dynamic...")
            }
        }
        stage('Publish') {
            steps {
                echo(message: "Publish...")
                
                script {
                    zip(
                        zipFile: "${ARTIFACT_NAME}",
                        archive: true,
                        dir: "pipeline",
                        glob: "")
                }
                 rtUpload(
                    serverId: 'Artifactory_1',
                    spec: """{
                        "files": [
                        {
                        "pattern": "${ARTIFACT_NAME}",
                        "target": "generic-local/release/neira/${env.BUILD_ID}/",
                        "flat" : "true"
                            
                        }
                        ]
                        
                    }"""
            )
            script {
                    if(params.FAIL_PIPELINE == true) {
                        error()
                    }
                }
           
            }
        }
    }
    post {
        failure {
            sendMail(env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, TEST_OUT)
        }
    }
}
