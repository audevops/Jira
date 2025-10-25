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
                        // Write workflow definition JSON to file
                        writeFile file: WORKFLOW_JSON, text: '''
{
  "name": "CI-CD Workflow",
  "description": "Continuous development pipeline: Dev → Test → Scan → Release → Approval → Deploy",
  "statuses": [
    {"name": "To Do"},
    {"name": "Development"},
    {"name": "Development Passed"},
    {"name": "Development Failed"},
    {"name": "Testing"},
    {"name": "Testing Passed"},
    {"name": "Testing Failed"},
    {"name": "Security/Code Scan"},
    {"name": "Scan Passed"},
    {"name": "Scan Failed"},
    {"name": "Release Preparation"},
    {"name": "Approval for Deployment"},
    {"name": "Deployed (Closed)"}
  ],
  "transitions": [
    {"name": "Start Development", "from": ["To Do"], "to": "Development", "description": "Developer begins work"},
    {"name": "Pass Review", "from": ["Development"], "to": "Development Passed", "description": "Code review success"},
    {"name": "Fail Review", "from": ["Development"], "to": "Development Failed", "description": "Review/test failed"},
    {"name": "Fix Issue", "from": ["Development Failed"], "to": "Development", "description": "Developer reworks issue"},
    {"name": "Move to Test", "from": ["Development Passed"], "to": "Testing", "description": "Code ready for testing"},
    {"name": "Pass", "from": ["Testing"], "to": "Testing Passed", "description": "All test cases pass"},
    {"name": "Fail", "from": ["Testing"], "to": "Testing Failed", "description": "Test cases fail"},
    {"name": "Fix", "from": ["Testing Failed"], "to": "Development", "description": "Send back for code fix"},
    {"name": "Move to Scan", "from": ["Testing Passed"], "to": "Security/Code Scan", "description": "Trigger scanning tools"},
    {"name": "Pass", "from": ["Security/Code Scan"], "to": "Scan Passed", "description": "No issues found"},
    {"name": "Fail", "from": ["Security/Code Scan"], "to": "Scan Failed", "description": "Vulnerabilities or issues found"},
    {"name": "Fix", "from": ["Scan Failed"], "to": "Development", "description": "Rework required"},
    {"name": "Prepare Release", "from": ["Scan Passed"], "to": "Release Preparation", "description": "Ready for release packaging"},
    {"name": "Request Approval", "from": ["Release Preparation"], "to": "Approval for Deployment", "description": "Submit for final sign-off"},
    {"name": "Approved", "from": ["Approval for Deployment"], "to": "Deployed (Closed)", "description": "Deployment done"},
    {"name": "Rejected", "from": ["Approval for Deployment"], "to": "Development", "description": "Rework required"}
  ],
  "draft": false
}
'''

                        echo "Creating JIRA Workflow..."
                        sh """
                            curl -s -X POST "${JIRA_URL}/rest/api/3/workflow" \
                              -u "${JIRA_USER}:${JIRA_TOKEN}" \
                              -H "Content-Type: application/json" \
                              -d @${WORKFLOW_JSON} \
                              -w '\\nHTTP Status: %{http_code}\\n'
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ JIRA Workflow created successfully on ${JIRA_URL}"
        }
        failure {
            echo "❌ Failed to create JIRA Workflow. Check API permissions or JSON format."
        }
    }
}

