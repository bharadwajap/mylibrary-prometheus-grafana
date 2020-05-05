#!groovy
def dockerAgent = 'docker-slave2'

def branch
def projectNamePrometh = 'mylibrary-prometheus'
def projectNameGrafana = 'mylibrary-grafana'

// pipeline
node(dockerAgent) {
    properties([
            [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator',daysToKeepStr: '1', numToKeepStr: '4']],
            parameters([
                	
			booleanParam(
				defaultValue: false,
				description: 'Deploy Prometheus?',
				name: 'deployPro'
			),
			booleanParam(
				defaultValue: false,
				description: 'Deploy Grafana?',
				name: 'deployGraf'
			)
            ])
    ])
	try {
		stage('Collect info') {
			checkout scm
			branch = env.BRANCH_NAME
		}

		stage('Dockerize') {
			final String activeContainers = sh(script: "sudo docker ps -a", returnStdout: true)
			boolean prometheusFound = activeContainers.toLowerCase().contains("${projectNamePrometh}")
			boolean grafanaFound = activeContainers.toLowerCase().contains("${projectNameGrafana}")
			if (prometheusFound && params.deployPro) {
				sh "sudo docker stop ${projectNamePrometh}"
				sh "sudo docker rm ${projectNamePrometh}"
			}
			if (grafanaFound && params.deployGraf) {
				sh "sudo docker stop ${projectNameGrafana}"
				sh "sudo docker rm ${projectNameGrafana}"
			}
			if(params.deployPro){
				sh "sudo docker build -t ${projectNamePrometh} ."
				sh "sudo docker run --network=host --restart=always --name=${projectNamePrometh} -td prom/prometheus --config.file=/etc/prometheus/prometheus.yml "
			}
			if(params.deployGraf){
				sh "sudo docker run --network=host --name=${projectNameGrafana} -td grafana/grafana"
			}
		}
	} catch (def e) {
		print "Exception occurred while running the pipeline"+ e
	} finally {
		deleteDir()
	}
}
