pipeline {
    agent any

    options {
      timestamps()
      disableConcurrentBuilds()
    }
    stages {
        stage('Patch pom.xml') {
            steps {
      //          dir("${env.REPO_DIR}") {
                sh """
                    set -e
                    echo "[INFO] Modifico il project.type nel pom.xml"
                    sed -i 's|<project.type>Platform</project.type>|<project.type>None</project.type>|' ./*.application/pom.xml
                    echo "[INFO] Controllo il risultato:"
                    grep "<project.type>" ./*.application/pom.xml
                """
                }
    //        }
        }

        stage('Build & Deploy Maven') {
            steps {
                dir("${env.REPO_DIR}") {
                sh """
                    set -e
                    POM_FILE_LOCATION=\$(find . -path "./*.parent/pom.xml")
                    echo "=== Start mvn clean ==="
                    mvn -V -B clean package -f \${POM_FILE_LOCATION}
                    echo "=== Start mvn deploy ==="
                    mvn -V -B deploy -f \${POM_FILE_LOCATION}
                """
                }
            }
        }
    }

    post {
        success {
            echo '=== Build effettuato con successo ==='
        }
        failure {
            echo 'Build fallita.'
        }
        always {
            echo "Branch: ${params.BRANCH}  |  Repo: ${params.REPO}"
        }
    }
}
