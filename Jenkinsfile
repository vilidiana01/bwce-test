pipeline {
    agent { label 'docker-ssh-jenkins-agent' }

    tools {
      maven 'mvn'
    }

    options {
      timestamps()
      disableConcurrentBuilds()
    }
    stages {
        stage('Patch pom.xml') {
            steps {
sh label: 'Patch pom.xml (solo cartella senza punto)', script: '''
bash -euo pipefail -lc '
  echo "[INFO] Individuo il pom.xml nella cartella senza punto..."
  mapfile -t targets < <(find . -maxdepth 2 -type f -name pom.xml -path "./*/pom.xml" ! -path "./*.*/*")

  if [ "${#targets[@]}" -ne 1 ]; then
    echo "[ERRORE] Atteso 1 solo pom.xml in cartella senza punto, trovati: ${#targets[@]}"
    printf "%s\\n" "${targets[@]}"
    exit 1
  fi

  target="${targets[0]}"
  echo "[INFO] File target: ${target}"

  echo "[INFO] Modifico il project.type nel pom.xml"
  sed -i '"'"'s|<project.type>Platform</project.type>|<project.type>None</project.type>|'"'"' "${target}"

  echo "[INFO] Controllo il risultato:"
  grep -n "<project.type>" "${target}"
'
'''

                }

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
