pipeline {
    agent any

    environment {
        JIRA_URL = "https://ausdevops.atlassian.net"
    }

    stages {
        stage('Validate JIRA Token') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jira_secret', usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_TOKEN')]) {
                    sh '''
                        echo "🔍 Validating JIRA credentials..."
                        curl -s -u "$JIRA_USER:$JIRA_TOKEN" \
                          -X GET "$JIRA_URL/rest/api/3/project" \
                          -H "Content-Type: application/json" \
                    '''
                }
            }
        }

        stage('Attempt Workflow Listing (Valid API)') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jira_secret', usernameVariable: 'JIRA_USER', passwordVariable: 'JIRA_TOKEN')]) {
                    sh '''
                        echo "📋 Fetching existing workflows..."
                        curl -s -u "$JIRA_USER:$JIRA_TOKEN" \
                          -X GET "$JIRA_URL/rest/api/3/workflow/search" \
                          -H "Accept: application/json" \
                          -w "\\nHTTP Status: %{http_code}\\n"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Jira API connection verified successfully."
        }
        failure {
            echo "❌ Failed to connect or execute API calls to Jira Cloud."
        }
    }
}

