pipeline {
    agent any

    environment {
        JIRA_URL = "https://ausdevops.atlassian.net"
        WORKFLOW_JSON = "workflow.json"
    }

    stages {
        stage('Create JIRA Workflow') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jira_secret', usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_TOKEN')]) {
                    script {
                        // Write workflow JSON to a file safely
                        writeFile file: WORKFLOW_JSON, text: '''
{
  "name": "CI-CD Workflow",
  "description": "Continuous development pipeline: Dev → Test → Scan → Release → Approval → Deploy",
  "statuses": [
    {"name": "To Do"},
    {"name": "In Progress"},
    {"name": "Done"}
  ]
}
'''

                        echo "Creating Jira Workflow..."
                        // Pass variables via environment, not Groovy string interpolation
                        sh """
                            curl -s -X POST "${JIRA_URL}/rest/api/3/workflow/create" \
                              -u "$JIRA_USER:$JIRA_TOKEN" \
                              -H "Content-Type: application/json" \
                              --data-binary @${WORKFLOW_JSON} \
                              -w '\\nHTTP Status: %{http_code}\\n'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ JIRA Workflow created successfully on ${env.JIRA_URL}"
        }
        failure {
            echo "❌ Failed to create JIRA Workflow. Check user permissions and JSON payload."
        }
    }
}
A
A
A
A
B
A
A
A

