pipeline {
  agent {
      label 'master'
  }
  stages {
    stage('Clean') {
      steps {
        sh 'rm -rf out'
      }
    }
    stage('Transform') {
      agent {
        docker {
          image 'cloudfluff/databaker'
          reuseNode true
        }
      }
      steps {
        sh "jupyter-nbconvert --to python --stdout tidy.ipynb | python"
      }
    }
    stage('Upload draftset') {
            steps {
                script {
                    def PMD = 'https://production-drafter-ons-alpha.publishmydata.com'
                    def credentials = 'onspmd'
                    def drafts = drafter.listDraftsets(PMD, credentials, 'owned')
                    def jobDraft = drafts.find  { it['display-name'] == env.JOB_NAME }
                    if (jobDraft) {
                        drafter.deleteDraftset(PMD, credentials, jobDraft.id)
                    }
                    def newJobDraft = drafter.createDraftset(PMD, credentials, env.JOB_NAME)
                    drafter.deleteGraph(PMD, credentials, newJobDraft.id,
                                        "http://gss-data.org.uk/graph/ons-trade-in-goods-mrets/metadata")
                    drafter.addData(PMD, credentials, newJobDraft.id,
                                    readFile("metadata/dataset.trig"), "application/trig")
                    def PIPELINE = 'http://production-grafter-ons-alpha.publishmydata.com/v1/pipelines'
                    runPipeline("${PIPELINE}/ons-table2qb.core/components/import",
                                newJobDraft.id, credentials, [[name: 'components-csv',
                                                                file: [name: 'metadata/components.csv', type: 'text/csv']]])
                    runPipeline("${PIPELINE}/ons-table2qb.core/data-cube/import",
                                newJobDraft.id, credentials, [[name: 'observations-csv',
                                                               file: [name: 'out/bop_observations.csv', type: 'text/csv']],
                                                              [name: 'dataset-name', value: 'ONS Trade in goods MRETS']])
                }
            }
        }
        stage('Test Draftset') {
            steps {
                echo 'Placeholder for acceptance tests from e.g. GDP-205'
            }
        }
        stage('Publish') {
            steps {
                script {
                    def PMD = 'https://production-drafter-ons-alpha.publishmydata.com'
                    def credentials = 'onspmd'
                    def drafts = drafter.listDraftsets(PMD, credentials, 'owned')
                    def jobDraft = drafts.find  { it['display-name'] == env.JOB_NAME }
                    if (jobDraft) {
                        drafter.publishDraftset(PMD, credentials, jobDraft.id)
                    } else {
                        error "Expecting a draftset for this job."
                    }
                }
            }
        }
    }
  }
  post {
    always {
      archiveArtifacts 'out/*'
    }
  }
}
