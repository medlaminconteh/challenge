// pipeline {
//     agent {
//         label 'buildertwo'
//     }

//     environment {
//         sonarqube_token = credentials('sonar-secrets-id')
//         IMAGE_NAME = "medlamin13956814/challenges"
//         IMAGE_TAG = "latest"
//     }
    
//     // tools {
//     //     maven 'Maven'
//     // //    jdk 'JDK11'
//     // }
    
//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }
        
//         // stage('Build') {
//         //     steps {
//         //         sh 'mvn clean package -DskipTests'
//         //     }
//         // }
        
//         // stage('Test') {
//         //     steps {
//         //         sh 'mvn test'
//         //     }
//         // }
        
//         // stage('SonarQube Analysis') {
//         //     steps {
//         //         withSonarQubeEnv('SonarQube') {
//         //             sh 'mvn sonar:sonar -Dsonar.projectKey=simple-java-maven-app -Dsonar.projectName="simple-java-maven-app"'
//         //         }
//         //     }
//         // }

//         stage('Build Docker Image') {
//             steps {
//                     sh 'sudo docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
//             }
//         }

//         //  Optional: Push Docker image to a registry
//         stage('Push Docker Image') {
//             steps {
//                 withCredentials([usernamePassword(
//                     credentialsId: 'docker-image-id',
//                     usernameVariable: 'DOCKER_USER',
//                     passwordVariable: 'DOCKER_PASS'
//                 )]) {
//                     sh '''
//                         echo "$DOCKER_PASS" | sudo docker login -u "$DOCKER_USER" --password-stdin
//                         sudo docker push ${IMAGE_NAME}:${IMAGE_TAG}
//                     '''
//                 }
//             }
//         }

//         // stage('Deploy Docker Image') {
//         //     steps {
//         //             sh 'sudo docker run ${IMAGE_NAME}:${IMAGE_TAG} -p 8888:8080'
//         //     }
//         // }


//         // stage('Build Docker Image') {
//         //     steps {
//         //         script {
//         //             docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
//         //         }
//         //     }
//         // }
        
//         // The good things at the end
//         // stage('Quality Gate') {
//         //     steps {
//         //         timeout(time: 1, unit: 'HOURS') {
//         //             waitForQualityGate abortPipeline: true
//         //         }
//         //     }
//         // }

//     }
// }




/////////
@Library('my_ocp_sharelib') _  
import com.aviro.OpenShiftHelper  

pipeline {
    //agent any  
    agent {
        label 'buildertwo'
    }
// oc login --token=sha256~xfssLZYgN179VUO7eaC1iJ1MfoUzd8LkG8F8c91IFKE --server=https://api.rm1.0a51.p1.openshiftapps.com:6443
    environment {
        OC_TOKEN = credentials('openshift_id')  // Jenkins credentials
        OC_SERVER = "https://api.rm1.0a51.p1.openshiftapps.com:6443"
        PROJECT = "lamr8-dev"
        IMAGE_NAME = "medlamin13956814/challenges:latest"
        APP_NAME = "web-terminal-tooling"
        APP_DEPLOYMENT = "workspace95ad191bab5b46c3"
    }

    stages {
        stage('Login to OpenShift') {
            steps {
                script {
                    OpenShiftHelper.login(this, OC_TOKEN, OC_SERVER)
                }
            }
        }


          stage('Trivy Security Scan') {
            steps {
                sh '''
                sudo docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                -v $PWD:/root/reports \
                aquasec/trivy image \
                --format template \
                --template "@/contrib/html.tpl" \
                -o /root/reports/trivy-report.html \
                ${IMAGE_NAME}
                '''
            }
        }

        stage('Publish Trivy Report') {
            steps {
                publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'trivy-report.html',
                reportName: 'Trivy Security Report',
                alwaysLinkToLastBuild: true
                ])
            }
        }


        // stage('Deploy Application') {
        //     steps {
        //         script {
        //             OpenShiftHelper.deploy(this, PROJECT, IMAGE, APP_NAME)
        //         }
        //     }
        // }

        stage('Deploy Application To Openshift') {
            steps {
                script {
                    //OpenShiftHelper.deployDeployment(this, PROJECT, IMAGE_NAME, APP_NAME, APP_DEPLOYMENT)
                    sh "oc project ${project}"
                    sh "oc new-app ${IMAGE_NAME} -name=${APP_NAME} --namespace=${PROJECT}"

                }
            }
        }

        stage('Check Deployment Status') {
            steps {
                script {
                    if (!OpenShiftHelper.checkDeployment(this, PROJECT, APP_NAME, APP_DEPLOYMENT)) {
                        error "Le déploiement a échoué !"
                    } else {
                        echo "L'application a été déployée avec succès sur OpenShifTt !"
                    }
                }
            }
        }
    }
}