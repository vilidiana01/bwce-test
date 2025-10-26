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
          steps{
          withCredentials([usernamePassword(credentialsId: 'jenkins', usernameVariable: 'GPR_USER', passwordVariable: 'GPR_TOKEN')]) {
          sh '''
            set -euo pipefail

            cat > settings.xml <<'EOF'
            <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
                      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
              <servers>
                <server>
                  <id>github</id>
                  <username>${env.GPR_USER}</username>
                  <password>${env.GPR_TOKEN}</password>
                </server>
              </servers>
              <profiles>
                <profile>
                  <id>github</id>
                  <repositories>
                    <repository>
                      <id>github</id>
                      <url>https://maven.pkg.github.com/vilidiana01/project-dependencies</url>
                    </repository>
                  </repositories>
                </profile>
              </profiles>
              <activeProfiles>
                <activeProfile>github</activeProfile>
              </activeProfiles>
            </settings>
            EOF

            # facoltativo: stampa versione Maven per diagnosi
            mvn -v

            # NB: specifica il pom giusto
            mvn -B -V -s settings.xml -f *.parent/pom.xml clean package
          '''
        }
      }
    }
    stage('Build docker image') {
        steps {
            sh '''
                    set -e
                    echo "[INFO] Build docker image using Dockerfile"
                    find . -type f -name '*.ear' -exec mv -t "$(pwd)" {} +
                    docker build -t test-app:${BUILD_NUMBER} -f Dockerfile .
                    echo "[INFO] Controllo il risultato:"
                    docker images
                '''
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
