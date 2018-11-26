pipeline {

    agent any

    stages {
        stage('Source Checkout') {
            agent { 
                docker { 
                    image 'pmaugeri/alpine-akamai-cli' 
                    args '--mount source=devops-lab,target=/root/data -u 0:0'
                }
            }
            steps {
                sh 'rm -rf /root/data/jenkins'
                sh 'mkdir -p /root/data/jenkins'
                sh 'git clone https://github.com/pmaugeri/ak-devops-lab.git /root/data/jenkins/ak-devops-lab.git'
            }
        }
        
        stage('Build') {
            agent { 
                docker { 
                    image 'pmaugeri/alpine-akamai-cli' 
                    args '--mount source=devops-lab,target=/root/data -u 0:0'
                }
            }
            steps {
                sh 'akamai property update --file /root/data/jenkins/ak-devops-lab.git/devops_lab1.master.json devops_lab1.qa'
                sh 'echo "[{ \"name\": \"PMUSER_ENV\", \"value\": \"QA\", \"description\": \"Possible values are: DEV, QA, PRD\", \"hidden\": false, \"sensitive\": false, \"action\": [ \"update\" ] }]" > /tmp/var.json'
                sh 'akamai property modify devops_lab1.qa --variables /tmp/var.json --addhosts devops-qa.restauring.com --edgehostname qa.devops.restauring.com.edgesuite.net'
            }
        }
        
        stage('Activate') {
            agent { 
                docker { 
                    image 'pmaugeri/alpine-akamai-cli' 
                    args '--mount source=devops-lab,target=/root/data -u 0:0'
                }
            }
            steps {
                echo "Activating configuration to Staging ..."
                sh 'akamai property activate devops_lab1.qa --network staging --email pmaugeri@akamai.com'
            }
        }

        stage('Unit Testing') {
            steps {
                echo "Unit testing in progress..."

                sh """
                docker run --name puppeteer --mount source=devops-lab,target=/root/data -u 0:0 pmaugeri/debian-puppeteer:version1 /usr/bin/nodejs /root/data/puppeteer/goto-akamai.com.js
                docker cp puppeteer:/root/data/puppeteer/www.akamai.com.png .
                docker rm -f puppeteer
                """
                archiveArtifacts artifacts: '*.png'
            }
        }

        stage('De-Activate') {
            agent { 
                docker { 
                    image 'pmaugeri/alpine-akamai-cli' 
                    args '--mount source=devops-lab,target=/root/data -u 0:0'
                }
            }
            steps {
                echo "De-activating configuration from Staging ..."
                sh 'akamai property deactivate devops_lab1.qa --network staging --email pmaugeri@akamai.com'
            }
        }
        
        stage('Cleanup') {
            agent { 
                docker { 
                    image 'pmaugeri/alpine-akamai-cli' 
                    args '--mount source=devops-lab,target=/root/data -u 0:0'
                }
            }
            steps {
                echo "Cleaning up ..."
                sh 'rm -rf /root/data/jenkins'
                //sh 'akamai property delete devops_lab1.qa'
            }
       }
    }

    post {
        failure {
            sh 'docker rm -f puppeteer'
        }
    }

    
}
