def getDevVersion() {
    script {
        return sh(script: "git rev-parse --short=7 HEAD ", returnStdout: true)
    }
}

def getBranchName() {
    script {
        return sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true)
    }
}


def buildlambda(dir) {
    if ("${BUILD_VERSION}" == "") {
        BUILD_VERSION = getDevVersion().trim()
    }
    sh "cd ${dir}/ && sam build "
}

def deploylambda(dir) {
    if ("${BUILD_VERSION}" == "") {
        BUILD_VERSION = getDevVersion().trim()
    }
    sh "cd ${dir}/ && sam deploy --no-fail-on-empty-changeset --template-file ${dir}/packaged-template.yml --config-file ${dir}/samconfig.${ENVIRONMENT}.toml --config-env ${ENVIRONMENT} --s3-bucket ${s3_bucket}"
    }
}

pipeline {
    parameters {
        text(name: 'services', defaultValue:'lambda1\nlambda2', description: 'services to build')
	string(name: 's3_bucket', defaultValue: 'cloudfront0307', description: 's3 bukcet for lambda'
	string(name: 'branch_name', defaultValue: 'main', description: ''
    }
    environment {
        BRANCH_NAME = "${params.branch_name}".trim()
        SERVICES = "${services}"
    }
    agent any
    stages {
        stage('Edit git config file'){
            steps{                
                sh 'git config --global url.ssh://git@github.com/.insteadOf https://github.com/'
            }
        }

        stage('Build Lambda') {
            steps {
                script {
                    services = "${params.services}".split('\n')
                    def branches = [:]
                    for (service in services) {
                        def svc = service
                        branches["Build ${service}"] = {buildlambda(svc)}
                    }
                    parallel branches
                }
            }
        }
        stage('Deploy Lambda') {
            steps {
                script {
                    services = "${params.services}".split('\n')
                    def branches = [:]
                    for (service in services) {
                        def svc = service
                        branches["Build ${service}"] = {deploylambda(svc)}
                    }
                    parallel branches
                }
            }
        }
    }
}