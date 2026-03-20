pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/DevOpsByNavin/room-booking.git'
        HARBOR_REGISTRY = 'harbor.nabinpoudel004.com.np'
        HARBOR_PROJECT = 'room-booking'
        HARBOR_USER = 'admin'
        EC2_HOST = '13.126.240.245'
        EC2_USER = 'admin'
        EC2_WORKDIR = 'room-booking'
    }

    stages {

        stage("Image Tagging") {
            steps {
                script {
                    env.FRONTEND_IMG = "${HARBOR_REGISTRY}/${HARBOR_PROJECT}/frontend:${BUILD_NUMBER}"
                    env.FRONTEND_LATEST = "${HARBOR_REGISTRY}/${HARBOR_PROJECT}/frontend:latest"
                }
            }
        }

        stage("Build image") {
            steps {
                sh '''
                    docker build -t "${FRONTEND_IMG}" -t "${FRONTEND_LATEST}" -f frontend/Dockerfile .
                '''
            }
        }

        stage("Push image to harbor") {
            steps {
                withCredentials([string(credentialsId: 'harbor', variable: 'HARBOR_API_KEY')]) {
                    sh '''
                        echo "${HARBOR_API_KEY}" | docker login "${HARBOR_REGISTRY}" --username "${HARBOR_USER}" --password-stdin

                        docker push "${FRONTEND_IMG}"
                        docker push "${FRONTEND_LATEST}"
                    '''
                }
            }
        }

        stage("Deploy images from server") {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-ec2-key', keyFileVariable: 'SSH_KEY')]) {
                script {
                    def remote = [
                        name: 'ec2',
                        host: env.EC2_HOST,
                        user: env.EC2_USER,
                        allowAnyHosts: true,
                        identityFile: SSH_KEY
                    ]

                    sshPut remote: remote, from: 'docker-compose.yml', into: "${env.EC2_WORKDIR}"

                    sshCommand remote: remote, command: """
                    cd "${env.EC2_WORKDIR}"
                    docker compose down --remove-orphans

                    cat > .env <<'EOF'
FRONTEND_IMG="${FRONTEND_IMG}"
EOF
                    docker compose up -d
                    docker image prune -af
                    """
                    }
                }

                // sshagent(credentials: ['deploy-ec2-key']) {
                //     sh """
                //         scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null docker-compose.yml ${EC2_USER}@${EC2_HOST}:${EC2_WORKDIR}/docker-compose.yml

                //         ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "${EC2_USER}@${EC2_HOST}" "
                //             cd ${EC2_WORKDIR}
                //             echo "FRONTEND_IMG=${FRONTEND_IMG}" > .env

                //             docker compose down --remove-orphans
                //             docker compose up -d
                //             docker image prune -fa
                //         "
                //     """
                // }
            }
        }
    }
}