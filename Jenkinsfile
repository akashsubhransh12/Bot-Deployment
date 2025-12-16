pipeline {
    agent any

    parameters {
        choice(
            name: 'PRODUCT',
            choices: ['Agoda', 'Booking', 'Expedia', 'MakeMyTrip', 'Goibibo'],
            description: 'Select product to deploy'
        )
        file(
            name: 'DLL_UPLOAD',
            description: 'Upload DLL from your local machine'
        )
    }

    environment {
        UPLOAD_DIR = 'D:\\Uploads'
    }

    stages {
        stage('Workspace Debug') {
            steps {
                echo "📂 Listing workspace files:"
                dir("${env.WORKSPACE}") {
                    bat 'dir'
                }
            }
        }

        stage('Validate DLL Upload') {
            steps {
                script {
                    def uploadedFile = "${env.WORKSPACE}\\${params.DLL_UPLOAD}"
                    if (!fileExists(uploadedFile)) {
                        error """
❌ NO DLL UPLOADED
The file '${params.DLL_UPLOAD}' does not exist in workspace!
👉 Make sure you upload the DLL using **Build with Parameters**
"""
                    }
                    echo "✅ DLL uploaded: ${uploadedFile}"
                    env.UPLOADED_DLL = uploadedFile
                }
            }
        }

        stage('Save DLL') {
            steps {
                script {
                    def targetDir  = "${env.UPLOAD_DIR}"
                    def targetFile = "${targetDir}\\${params.DLL_UPLOAD}"

                    powershell """
                        if (!(Test-Path '${targetDir}')) {
                            New-Item -ItemType Directory -Path '${targetDir}' -Force | Out-Null
                        }
                        Copy-Item '${env.UPLOADED_DLL}' '${targetFile}' -Force
                    """
                    echo "✅ DLL copied to upload folder: ${targetFile}"
                }
            }
        }

        stage('Confirm DLL') {
            steps {
                input message: "Confirm the DLL is correct: ${params.DLL_UPLOAD}", ok: 'Proceed'
            }
        }

        stage('Deploy DLL') {
            steps {
                script {
                    def servers = [
                        Agoda   : [ip:'10.10.10.54', cred:'agoda-creds', dest:'C:\\RankBot\\Agoda'],
                        Booking : [ip:'10.10.10.63', cred:'booking-creds', dest:'C:\\RankBot\\Booking'],
                        Expedia : [ip:'10.10.10.XX', cred:'expedia-creds', dest:'C:\\RankBot\\Expedia']
                        // Add other servers...
                    ]

                    def cfg = servers[params.PRODUCT]
                    if (!cfg || cfg.ip.contains('XX')) {
                        error "❌ Server not configured for ${params.PRODUCT}"
                    }

                    withCredentials([
                        usernamePassword(
                            credentialsId: cfg.cred,
                            usernameVariable: 'DEPLOY_USER',
                            passwordVariable: 'DEPLOY_PASS'
                        )
                    ]) {
                        powershell """
                        \$sec = ConvertTo-SecureString \$env:DEPLOY_PASS -AsPlainText -Force
                        \$cred = New-Object System.Management.Automation.PSCredential(\$env:DEPLOY_USER, \$sec)
                        \$s = New-PSSession -ComputerName ${cfg.ip} -Credential \$cred
                        Copy-Item '${env.UPLOAD_DIR}\\${params.DLL_UPLOAD}' '${cfg.dest}' -ToSession \$s -Force
                        Remove-PSSession \$s
                        """
                    }
                }
            }
        }
    }

    post {
        success { echo '✅ DEPLOYMENT SUCCESSFUL' }
        failure { echo '❌ DEPLOYMENT FAILED' }
        always { echo '🧹 Pipeline finished' }
    }
}

