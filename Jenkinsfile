pipeline {
    agent any

    parameters {
        string(name: 'JIRA_URL', defaultValue: 'https://ausdevops.atlassian.net', description: 'Enter your Jira Cloud base URL')
        string(name: 'JIRA_TICKET', defaultValue: 'CICD-3', description: 'Enter the Jira issue key (e.g. CICD-123)')
        choice(name: 'ISSUE_TYPE', choices: ['Story', 'Task', 'Epic'], description: 'Select the Jira issue type')
    }

    environment {
        JIRA_URL = "${params.JIRA_URL}"
        JIRA_TICKET = "${params.JIRA_TICKET}"
        ISSUE_TYPE = "${params.ISSUE_TYPE}"
    }

    stages {
        stage('Validate JIRA Token') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jira_secret', usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_TOKEN')]) {
                    sh '''
                        echo "🔍 Validating Jira credentials..."
                        curl -s -u "$JIRA_USER:$JIRA_TOKEN" \
                          -X GET "$JIRA_URL/rest/api/3/myself" \
                          -H "Accept: application/json" 
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "🏗️ Running Build Stage..."
                    // simulate a build process
                    sh 'sleep 2'
                }
            }
            post {
                success {
                    updateJiraComment("✅ Build stage passed successfully.")
                }
                failure {
                    updateJiraComment("❌ Build stage failed.")
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "�� Running Test Stage..."
                    // simulate tests
                    sh 'sleep 2'
                }
            }
            post {
                success {
                    updateJiraComment("✅ Test stage passed successfully.")
                }
                failure {
                    updateJiraComment("❌ Test stage failed.")
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "🚀 Running Deploy Stage..."
                    // simulate deploy
                    sh 'sleep 2'
                }
            }
            post {
                success {
                    updateJiraComment("✅ Deploy stage passed successfully.")
                }
                failure {
                    updateJiraComment("❌ Deploy stage failed.")
                }
            }
        }
    }

    post {
        always {
            script {
                echo "📝 Updating Jira ticket with final status..."
                def status = currentBuild.result ?: 'SUCCESS'
                def finalMessage = (status == 'SUCCESS') ?
                    "🎉 All stages completed successfully in Jenkins pipeline for ${ISSUE_TYPE} ${JIRA_TICKET}" :
                    "⚠️ Jenkins pipeline encountered failures for ${ISSUE_TYPE} ${JIRA_TICKET}. Please review failed stages."

                updateJiraComment(finalMessage)
            }
        }
    }
}

//
// ---- Shared Jira Update Function ----
//
def updateJiraComment(String message) {
    withCredentials([usernamePassword(credentialsId: 'jira_secret', usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_TOKEN')]) {
        sh(
            script: """
                #!/bin/bash
                set -e

                echo "🧩 Updating comment on Jira issue ${env.JIRA_TICKET}..."

                # Construct the ADF-compliant JSON payload
                COMMENT_PAYLOAD=\$(cat <<EOF
{
  "body": {
    "type": "doc",
    "version": 1,
    "content": [
      {
        "type": "paragraph",
        "content": [
          {
            "type": "text",
            "text": "${message.replaceAll('"', '\\"')}"
          }
        ]
      }
    ]
  },
  "visibility": {
    "type": "role",
    "value": "Administrators"
  }
}
EOF
)

                # Make the API call
                curl -s -u "$JIRA_USER:$JIRA_TOKEN" \\
                  -X PUT "${env.JIRA_URL}/rest/api/3/issue/${env.JIRA_TICKET}/comment/10000" \\
                  -H "Accept: application/json" \\
                  -H "Content-Type: application/json" \\
                  -d "\$COMMENT_PAYLOAD" \\
                  -w "\\nHTTP Status: %{http_code}\\n"
            """,
            returnStatus: true
        )
    }
}

