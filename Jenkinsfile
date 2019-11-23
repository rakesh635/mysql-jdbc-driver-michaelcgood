pipeline {
    agent any
    stages {
        stage('checkout') {
            steps {
                echo 'checkout'
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/rakesh635/mysql-jdbc-driver-michaelcgood.git']]])
            }
        }
        stage('Build') {
            steps {
                echo 'Clean Build'
                    sh "ls"
                    sh "pwd"
                    sh "mvn sonar:sonar clean compile -Dtest=\\!TestRunner* -DfailIfNoTests=false -Dsonar.projectKey=testproj1_rakesh -Dsonar.organization=rakesh635-github -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=b6011a743e95e30948765c87a16ee9b8cac28712"  
                    //sh 'mvn clean compile'
            }
        }
        stage('Package') {
            steps {
                echo 'Packaging'
                sh 'mvn package -DskipTests'
            }
        }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]//,
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                /*[artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]*/
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'curl --upload-file target/mysql-jdbc-0.0.1-SNAPSHOT.jar "http://tomcat:password@35.200.135.60:8081/manager/text/deploy?path=/hello1&update=true"'
                //withCredentials([usernamePassword(credentialsId: 'nexusadmin', passwordVariable: 'pass', usernameVariable: 'user')]) {
                //    sh 'curl --upload-file target/hello-world-war-1.0.0-SNAPSHOT.war "http://${user}:${pass}@34.93.240.217:8082/manager/text/deploy?path=/hello&update=true"'
                //}
            }
        }
        stage('Test') {
            steps {
                echo 'Testing - Dummy Stage'
                //sh 'mvn test'
            }
        }
        stage('Artifact Promotion') {
            steps {
                artifactPromotion (
                    promoterClass: 'org.jenkinsci.plugins.artifactpromotion.NexusOSSPromotor',
                debug: false,
                    groupId: 'com.edurekademo.tutorial',
                    artifactId: 'addressbook',
                    version: '2.0-SNAPSHOT',
                    extension: 'war',
                    //stagingRepository: 'http://nexus.myorg.com:8080/content/repositories/release-candidates',
                    stagingRepository: 'http://34.93.7.213:8081/repository/maven-snapshots',
                    stagingUser: 'admin',
                    stagingPW: 'admin123',
                    skipDeletion: true,
                    releaseRepository: 'http://34.93.7.213:8081/repository/maven-releases',
                    releaseUser: 'admin',
                    releasePW: 'admin123'
                )
            }
        }
    }
    tools {
        maven 'maven3.3.9'
        jdk 'openjdk8'
    }
    environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "35.244.49.94:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "maven-snapshots"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexusadmin"
    }
    
    post {
        always {
            echo 'JENKINS PIPELINE'
        }
        success {
            echo 'JENKINS PIPELINE SUCCESSFUL'
        }
        failure {
            echo 'JENKINS PIPELINE FAILED'
        }
        unstable {
            echo 'JENKINS PIPELINE WAS MARKED AS UNSTABLE'
        }
        changed {
            echo 'JENKINS PIPELINE STATUS HAS CHANGED SINCE LAST EXECUTION'
        }
    }
}
