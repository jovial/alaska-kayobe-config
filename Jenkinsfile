def checkConfigOption(String option, Class<?> type) {
    if (!config.containsKey(option)) {
        error("${option} is a a required configuration option")
    }
    if (!type.isInstance(config[option])) {
        error("The configuration option, ${option}, should be of type: ${type}")
    }
}

pipeline {
    options { disableConcurrentBuilds() }
    agent { label 'docker' }
    parameters {
        string description: 'Command to run in docker container', name: 'COMMAND', trim: false
    }
    environment {
        REGISTRY = "${config.docker_registry}"
        KAYOBE_IMAGE = "kayobe-config:${env.GIT_COMMIT}"
    }

    stages {
        // Do parameter validation in a stage so that the git checkout of the pipeline still works.
        // This is necessary to update the build parameters.
        stage('Validate parameters') {
            steps {
                script {
                    if (!params.COMMAND){
                        // Fail early
                        error("You must set the COMMAND parameter")
                    }
                }
                script {
                    def config = readYaml file: 'jenkins/config.yml'
                    checkConfigOption("docker_registry", String)
                    checkConfigOption("kayobe_ssh_creds", String)
                    checkConfigOption("kayobe_vault_password", String)
                }
            }

        }
        stage('Build and Push') {
            steps {
                script {
                    def kayobeImage = docker.build("$KAYOBE_IMAGE")
                    docker.withRegistry("$REGISTRY") {
                        kayobeImage.push()
                        kayobeImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy') {
            stages {
                stage('Prepare Secrets') {
                    environment {
                        KAYOBE_VAULT_PASSWORD = credentials("${config.kayobe_vault_password}")
                        KAYOBE_SSH_CREDS_FILE = credentials("${config.kayobe_ssh_creds}")
                    }
                    steps {
                        sh 'mkdir -p secrets/.ssh'
                        sh "cp $KAYOBE_SSH_CREDS_FILE secrets/.ssh/id_rsa"
                        sh(returnStdout: false, script: 'ssh-keygen -y -f secrets/.ssh/id_rsa > secrets/.ssh/id_rsa.pub')
                        sh(returnStdout: false, script: 'echo $KAYOBE_VAULT_PASSWORD > secrets/vault.pass')
                    }
                }
                stage('Optionally prepare SSH Config') {
                    when {
                        expression { config.kayobe_ssh_config != null && config.kayobe_ssh_config != '' }
                    }
                    environment {
                        KAYOBE_SSH_CONFIG_FILE = credentials("${config.kayobe_ssh_config}")
                    }
                    steps {
                        sh "cp $KAYOBE_SSH_CONFIG_FILE secrets/.ssh/config"
                    }
                }
                stage('Run Kayobe') {
                    agent {
                        docker {
                            image "$KAYOBE_IMAGE"
                            registryUrl "$REGISTRY"
                            reuseNode true
                        }
                    }
                    environment {
                        KAYOBE_VAULT_PASSWORD = credentials("${config.kayobe_vault_password}")
                    }
                    steps {
                        sh 'cp -R secrets/. /secrets'
                        sh '/bin/entrypoint.sh echo READY'
                        sh "${params.COMMAND}"
                    }
                }
            }
        }
    }
    post {
        cleanup {
            dir('secrets') {
                deleteDir()
            }
        }
   }
}
