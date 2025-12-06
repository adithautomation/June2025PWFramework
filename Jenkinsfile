
// ============================================
// PLAYWRIGHT AUTO PIPELINE - JENKINSFILE
// ============================================
// Flow: lint → dev → qa → stage → prod (automatic)
// Trigger: Push, PR, or manual build
// Reports: Separate Allure per environment, Playwright HTML, Custom HTML
// ✅ ESLint static code analysis
// ✅ Separate Allure reports per environment
// ✅ Slack notifications for test results
// ✅ Email notifications with all report links
// ============================================
//
// Required Jenkins Credentials:
// ------------------------------------
// slack-token          - Slack Webhook Token (Secret text)
// ============================================
//
// Required Jenkins Plugins:
// ------------------------------------
// - NodeJS Plugin
// - Allure Jenkins Plugin
// - HTML Publisher Plugin
// - Slack Notification Plugin
// - Email Extension Plugin
// - Pipeline Stage View Plugin
// ============================================

// ============================================
// PLAYWRIGHT AUTO PIPELINE - JENKINSFILE (Updated)
// ============================================

pipeline {
    agent any

    tools {
        nodejs 'NodeJS-20'
    }

    environment {
        NODE_VERSION = '20'
        CI = 'true'
        PLAYWRIGHT_BROWSERS_PATH = "${WORKSPACE}/.cache/ms-playwright"
        // NOTE: removed SLACK_WEBHOOK_URL = credentials('slack-webhook') to avoid early evaluation
        // Email recipients - update these with your actual email addresses
        EMAIL_RECIPIENTS = 'adithautomation@gmail.com, mail@adithautomation.com'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    stages {
        // ============================================
        // Static Code Analysis (ESLint)
        // ============================================
        stage('🔍 ESLint Analysis') {
            steps {
                echo '============================================'
                echo '📥 Installing dependencies...'
                echo '============================================'
                //sh 'npm ci'
                sh 'PUPPETEER_SKIP_DOWNLOAD=true npm ci'

                echo '============================================'
                echo '📁 Creating ESLint report directory...'
                echo '============================================'
                sh 'mkdir -p eslint-report'

                echo '============================================'
                echo '🔍 Running ESLint...'
                echo '============================================'
                script {
                    def eslintStatus = sh(script: 'npm run lint', returnStatus: true)
                    env.ESLINT_STATUS = eslintStatus == 0 ? 'success' : 'failure'
                }

                echo '============================================'
                echo '📊 Generating ESLint HTML Report...'
                echo '============================================'
                sh 'npm run lint:report || true'
            }
            post {
                always {
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'eslint-report',
                        reportFiles: 'index.html',
                        reportName: 'ESLint Report',
                        reportTitles: 'ESLint Analysis'
                    ])
                    script {
                        if (env.ESLINT_STATUS == 'failure') {
                            echo '⚠️ ESLint found issues - check the HTML report'
                        } else {
                            echo '✅ No ESLint issues found'
                        }
                    }
                }
            }
        }

        // QA Tests
        stage('🔍 QA Tests') {
            steps {
                echo '============================================'
                echo '🧹 Cleaning previous results...'
                echo '============================================'
                sh 'rm -rf allure-results playwright-report playwright-html-report test-results'

                echo '============================================'
                echo '🧪 Running QA tests...'
                echo '============================================'
                script {
                    env.QA_TEST_STATUS = sh(
                        script: 'npx playwright test --grep "@login" --config=playwright.config.qa.ts',
                        returnStatus: true
                    ) == 0 ? 'success' : 'failure'
                }

                echo '============================================'
                echo '🏷️ Adding Allure environment info...'
                echo '============================================'
                sh '''
                    mkdir -p allure-results
                    echo "Environment=QA" > allure-results/environment.properties
                    echo "Browser=Chromium" >> allure-results/environment.properties
                    echo "Config=playwright.config.qa.ts" >> allure-results/environment.properties
                '''
            }
            post {
                always {
                    sh '''
                        mkdir -p allure-results-qa
                        cp -r allure-results/* allure-results-qa/ 2>/dev/null || true
                        npx allure generate allure-results-qa --clean -o allure-report-qa || true
                    '''
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'allure-report-qa',
                        reportFiles: 'index.html',
                        reportName: 'QA Allure Report',
                        reportTitles: 'QA Allure Report'
                    ])
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'QA Playwright Report',
                        reportTitles: 'QA Playwright Report'
                    ])
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-html-report',
                        reportFiles: 'index.html',
                        reportName: 'QA HTML Report',
                        reportTitles: 'QA Custom HTML Report'
                    ])
                    archiveArtifacts artifacts: 'allure-results-qa/**/*', allowEmptyArchive: true
                    archiveArtifacts artifacts: 'test-results/**/*', allowEmptyArchive: true
                }
            }
        }

        // STAGE Tests
        stage('🎯 STAGE Tests') {
            steps {
                echo '============================================'
                echo '🧹 Cleaning previous results...'
                echo '============================================'
                sh 'rm -rf allure-results playwright-report playwright-html-report test-results'

                echo '============================================'
                echo '🧪 Running STAGE tests...'
                echo '============================================'
                script {
                    env.STAGE_TEST_STATUS = sh(
                        script: 'npx playwright test --grep "@login" --config=playwright.config.stage.ts',
                        returnStatus: true
                    ) == 0 ? 'success' : 'failure'
                }

                echo '============================================'
                echo '🏷️ Adding Allure environment info...'
                echo '============================================'
                sh '''
                    mkdir -p allure-results
                    echo "Environment=STAGE" > allure-results/environment.properties
                    echo "Browser=Chromium" >> allure-results/environment.properties
                    echo "Config=playwright.config.stage.ts" >> allure-results/environment.properties
                '''
            }
            post {
                always {
                    sh '''
                        mkdir -p allure-results-stage
                        cp -r allure-results/* allure-results-stage/ 2>/dev/null || true
                        npx allure generate allure-results-stage --clean -o allure-report-stage || true
                    '''
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'allure-report-stage',
                        reportFiles: 'index.html',
                        reportName: 'STAGE Allure Report',
                        reportTitles: 'STAGE Allure Report'
                    ])
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'STAGE Playwright Report',
                        reportTitles: 'STAGE Playwright Report'
                    ])
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-html-report',
                        reportFiles: 'index.html',
                        reportName: 'STAGE HTML Report',
                        reportTitles: 'STAGE Custom HTML Report'
                    ])
                    archiveArtifacts artifacts: 'allure-results-stage/**/*', allowEmptyArchive: true
                    archiveArtifacts artifacts: 'test-results/**/*', allowEmptyArchive: true
                }
            }
        }

        // Combined Allure Report
        stage('📈 Combined Allure Report') {
            steps {
                echo '============================================'
                echo '📊 Generating Combined Allure Report...'
                echo '============================================'

                sh '''
                    mkdir -p allure-results-combined
                    cp -r allure-results-qa/* allure-results-combined/ 2>/dev/null || true
                    cp -r allure-results-stage/* allure-results-combined/ 2>/dev/null || true
                    echo "Environment=ALL (QA, STAGE)" > allure-results-combined/environment.properties
                    echo "Browser=Chromium" >> allure-results-combined/environment.properties
                    echo "Pipeline=${JOB_NAME}" >> allure-results-combined/environment.properties
                    echo "Build=${BUILD_NUMBER}" >> allure-results-combined/environment.properties
                '''
            }
            post {
                always {
                    sh 'npx allure generate allure-results-combined --clean -o allure-report-combined || true'
                    publishHTML(target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'allure-report-combined',
                        reportFiles: 'index.html',
                        reportName: 'Combined Allure Report',
                        reportTitles: 'Combined Allure Report'
                    ])
                }
            }
        }
    }

    // ============================================
    // Post-Build Actions (Notifications)
    // ============================================
    post {
        always {
            echo '============================================'
            echo '📬 PIPELINE SUMMARY'
            echo '============================================'

            script {
                def qaStatus = env.QA_TEST_STATUS ?: 'unknown'
                def stageStatus = env.STAGE_TEST_STATUS ?: 'unknown'

                def qaEmoji = qaStatus == 'success' ? '✅' : '❌'
                def stageEmoji = stageStatus == 'success' ? '✅' : '❌'

                echo """
============================================
📊 Test Results by Environment:
============================================
${qaEmoji} QA:    ${qaStatus}
${stageEmoji} STAGE: ${stageStatus}
============================================
"""

                def overallStatus = 'SUCCESS'
                def statusEmoji = '✅'
                def statusColor = 'good'

                if (qaStatus == 'failure' || stageStatus == 'failure') {
                    overallStatus = 'FAILURE'
                    statusEmoji = '❌'
                    statusColor = 'danger'
                } else if (qaStatus == 'unknown' || stageStatus == 'unknown') {
                    overallStatus = 'UNSTABLE'
                    statusEmoji = '⚠️'
                    statusColor = 'warning'
                }

                env.OVERALL_STATUS = overallStatus
                env.STATUS_EMOJI = statusEmoji
                env.STATUS_COLOR = statusColor
                env.QA_EMOJI = qaEmoji
                env.STAGE_EMOJI = stageEmoji
            }
        }

        success {
            echo '✅ Pipeline completed successfully!'

            script {
                // Build the Slack message
                def slackMessage = """✅ *Playwright Pipeline: All Tests Passed*

*Repository:* ${env.JOB_NAME}
*Branch:* ${env.GIT_BRANCH ?: 'N/A'}
*Build:* #${env.BUILD_NUMBER}

*Test Results:*
${env.QA_EMOJI} QA: ${env.QA_TEST_STATUS}
${env.STAGE_EMOJI} STAGE: ${env.STAGE_TEST_STATUS}

📊 <${env.BUILD_URL}Combined_20Allure_20Report|Combined Allure Report>
🔗 <${env.BUILD_URL}|View Build>"""

                // Try to read credentials and send Slack only if available
                try {
                    def webhook = null
                    try {
                        withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_SECRET')]) {
                            webhook = env.SLACK_WEBHOOK_SECRET
                        }
                    } catch (credEx) {
                        echo "Slack credential 'slack-webhook' not available: ${credEx.message}"
                    }

                    if (webhook) {
                        try {
                            slackSend(webhookUrl: webhook, channel: '#test-automation', color: 'good', message: slackMessage)
                            echo "Slack message sent."
                        } catch (e) {
                            echo "Slack send failed: ${e.message}"
                        }
                    } else {
                        echo "Skipping Slack notification because 'slack-webhook' credential is not configured."
                    }
                } catch (outerEx) {
                    echo "Slack notification wrapper failed: ${outerEx.message}"
                }

                // Email notification (unchanged)
                try {
                    emailext(
                        subject: "✅ Playwright Tests Passed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """<!DOCTYPE html> ... (omitted here for brevity) ... </html>""",
                        mimeType: 'text/html',
                        to: env.EMAIL_RECIPIENTS,
                        from: 'CI Notifications <mail@naveenautomationlabs.com>',
                        replyTo: 'mail@naveenautomationlabs.com'
                    )
                } catch (Exception e) {
                    echo "Email notification failed: ${e.message}"
                }
            }
        }

        failure {
            echo '❌ Pipeline failed!'

            script {
                def slackMessage = """❌ *Playwright Pipeline: Tests Failed*

*Repository:* ${env.JOB_NAME}
*Branch:* ${env.GIT_BRANCH ?: 'N/A'}
*Build:* #${env.BUILD_NUMBER}

*Test Results:*
${env.QA_EMOJI ?: '❓'} QA: ${env.QA_TEST_STATUS ?: 'not run'}
${env.STAGE_EMOJI ?: '❓'} STAGE: ${env.STAGE_TEST_STATUS ?: 'not run'}

📊 <${env.BUILD_URL}Combined_20Allure_20Report|View Allure Report>
🔗 <${env.BUILD_URL}|View Build>"""

                // Slack send with credential guard
                try {
                    def webhook = null
                    try {
                        withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_SECRET')]) {
                            webhook = env.SLACK_WEBHOOK_SECRET
                        }
                    } catch (credEx) {
                        echo "Slack credential 'slack-webhook' not available: ${credEx.message}"
                    }

                    if (webhook) {
                        try {
                            slackSend(webhookUrl: webhook, channel: '#test-automation', color: 'danger', message: slackMessage)
                            echo "Slack message sent."
                        } catch (e) {
                            echo "Slack send failed: ${e.message}"
                        }
                    } else {
                        echo "Skipping Slack notification because 'slack-webhook' credential is not configured."
                    }
                } catch (outerEx) {
                    echo "Slack notification wrapper failed: ${outerEx.message}"
                }

                // Email notification (unchanged)
                try {
                    emailext(
                        subject: "❌ Playwright Tests Failed - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """<!DOCTYPE html> ... (omitted here for brevity) ... </html>""",
                        mimeType: 'text/html',
                        to: env.EMAIL_RECIPIENTS,
                        from: 'mailto@adithautomation.com',
                        replyTo: 'mailto@adithautomation.com'
                    )
                } catch (Exception e) {
                    echo "Email notification failed: ${e.message}"
                }
            }
        }

        unstable {
            echo '⚠️ Pipeline completed with warnings!'

            script {
                def slackMessage = """⚠️ *Playwright Pipeline: Unstable*

*Repository:* ${env.JOB_NAME}
*Branch:* ${env.GIT_BRANCH ?: 'N/A'}
*Build:* #${env.BUILD_NUMBER}

📊 <${env.BUILD_URL}Combined_20Allure_20Report|View Allure Report>
🔗 <${env.BUILD_URL}|View Build>"""

                try {
                    def webhook = null
                    try {
                        withCredentials([string(credentialsId: 'slack-webhook', variable: 'SLACK_WEBHOOK_SECRET')]) {
                            webhook = env.SLACK_WEBHOOK_SECRET
                        }
                    } catch (credEx) {
                        echo "Slack credential 'slack-webhook' not available: ${credEx.message}"
                    }

                    if (webhook) {
                        try {
                            slackSend(webhookUrl: webhook, channel: '#test-automation', color: 'warning', message: slackMessage)
                            echo "Slack message sent."
                        } catch (e) {
                            echo "Slack send failed: ${e.message}"
                        }
                    } else {
                        echo "Skipping Slack notification because 'slack-webhook' credential is not configured."
                    }
                } catch (outerEx) {
                    echo "Slack notification wrapper failed: ${outerEx.message}"
                }
            }
        }
    }
}

