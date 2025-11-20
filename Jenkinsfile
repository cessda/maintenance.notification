/*
	Copyright CESSDA ERIC 2023

	Licensed under the Apache License, Version 2.0 (the "License"); you may not
	use this file except in compliance with the License.
	You may obtain a copy of the License at
	https://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.
*/
pipeline{

	options {
		disableConcurrentBuilds()
	}

	environment {
		productName = 'maintanence'
		imageTag = "${DOCKER_ARTIFACT_REGISTRY}/${productName}:${GIT_COMMIT}"
		PATH = "${tool 'helm'}:${PATH}"
	}

	agent any

	stages {
		stage('Setup GCP Environment') {
			steps {
				sh "gcloud config set project ${GCP_PROJECT}"
				sh "gcloud container clusters get-credentials production-cluster --zone=${zone}"
			}
			when { branch 'main' }
		}
		stage('Build Docker Container') {
			steps {
				dir('container') {
					sh "docker build -t ${imageTag} ."
				}
			}
		}
		stage('Push Docker Image') {
			steps {
				sh "gcloud auth configure-docker"
				sh "docker push ${imageTag}"
			}
			when { branch 'main' }
		}
		stage('Deploy Maintanence Notice') {
			steps {
				sh "helm upgrade maintanence ./chart --namespace cessda-mgmt --create-namespace --install" + 
					" --set image.tag=${GIT_COMMIT}"
			}
			when { branch 'main' }
		}
		stage('Validate Kubernetes Manifests') {
			steps {
				sh "helm template maintanence ./chart" + 
					" --set image.tag=${env.BRANCH_NAME}-${env.BUILD_NUMBER}" +
					" | kubectl create --validate=true --dry-run=client -f -"
			}
			when { not { branch 'main'} }
		}
	}
}