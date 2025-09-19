pipeline {
    agent any
    parameters {
        string(name: 'SERVICE', description: 'Docker Composer Service Name', defaultValue: 'qbittorrent')
        string(name: 'WORKING_DIR', description: 'Docker Composer Working Directory', defaultValue: '/home/aform/ezarr')
        string(name: 'SSH_HOST', description: 'SSH Host Where Docker Composer Runs', defaultValue: '192.168.100.5')
        string(name: 'SSH_USER', description: 'SSH User', defaultValue: 'aform')
        string(name: 'CREDENTIALS_ID', description: 'SSH Credentials ID', defaultValue: 'ubuntu-vm-on-T580')
    }
    environment {
        SERVICE                     = "${params.SERVICE}"
        WORKING_DIR                 = "${params.WORKING_DIR}"
        SSH_HOST                    = "${params.SSH_HOST}"
        SSH_USER                    = "${params.SSH_USER}"
        CREDENTIALS_ID              = "${params.CREDENTIALS_ID}"
    }
    stages {
        stage('On Jenkins Host') {
            stages {
                stage('Verify Nginx Config') {
                    steps {
                        sshagent(credentials: [CREDENTIALS_ID]) {
                            def remoteScript = """
                            echo "!!!=> Change Working Directory to: ${WORKING_DIR}"
                            echo =============================================
                            cd ${WORKING_DIR}
                            service=${SERVICE}

                            echo "!!!=> Information about current image"
                            echo =============================================
                            imageId=\$(docker compose images --format json \${service} | jq -r '.[0].ID')
                            imageDigest=\$(docker images --digests  --format json --no-trunc | jq --indent 2 --arg v "\${imageId}" -c 'select(.ID == \$v)')
                            echo "\${imageDigest}"
                            echo =============================================

                            echo "!!!=> Try to pull latest image(s)"
                            echo =============================================
                            docker compose pull \${service}
                            echo =============================================

                            echo "!!!=> Force recreate the image with latest image"
                            echo =============================================
                            docker compose up -d --no-deps --force-recreate --build \${service}
                            echo ============================================= 

                            echo "!!!=> Information about new image"
                            echo =============================================
                            imageId=\$(docker compose images --format json \${service} | jq -r '.[0].ID')
                            imageDigest=\$(docker images --digests  --format json --no-trunc | jq --indent 2 --arg v "\${imageId}" -c 'select(.ID == \$v)')
                            echo "\${imageDigest}"
                            echo =============================================
                            """.stripIndent().trim()

                            def sshCommand = """
                                ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} 'bash -s' << 'EOF'
                                ${remoteScript}
    EOF
            """.stripIndent().trim()

                            echo "Executing SSH command:"
                            echo "${sshCommand}"

                            sh sshCommand
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                currentBuild.description = "${params.SERVICE} on ${params.SSH_HOST}"
            }
        }
    }    
}