pipeline {
  agent none

  triggers {
      bitbucketPush()
      cron('H 3 * * *')
  }

  options {
      buildDiscarder(
          logRotator(artifactNumToKeepStr: '5')
      )
  }

  stages {
    stage("test") {

      matrix {
        axes {
          axis {
            name 'version'
            values '1.21.10', '1.22.7', '1.23.5'
          }
        }

        stages {
          stage('deployment') {
            options {
              lock label: 'kind',
                   quantity: 1,
                   inversePrecedence: true
            }

            agent {
              dockerfile {
                label 'docker'
                dir 'test'
                args '-v /var/run/docker.sock:/var/run/docker.sock \
                -v /etc/passwd:/etc/passwd:ro \
                -e HOME=${workspace} \
                --group-add 999'
              }
            }

            steps {
              sh """
                kind create cluster --name='mok-${version}' --config='kind.yaml' --image='kindest/node:v${version}'
                helm install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx
                helm install eck-operator eck-operator --version '1.9.1' --repo https://helm.elastic.co
                kubectl wait --timeout=120s --for=condition=available --all deployments

                skaffold run --profile="ci"
                kubectl wait --timeout=3600s --for=condition=complete job/cypress
                kubectl get pods --all-namespaces
              """
            } // steps

            post {
              always {
                sh """
                  kubectl get pods,pvc,deployments,statefulsets,jobs --all-namespaces
                  kubectl describe node
                  kubectl logs job/cypress
                """
              } // always

              cleanup {
                sh """
                  kind delete cluster --name='mok-${version}'
                """
              } // cleanup
            } // post
          } // stage "deployment"
        } // stages
      } // matrix
    } // stage "test"
  } // stages
} // pipeline
