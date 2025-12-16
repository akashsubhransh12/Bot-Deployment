
pipeline {
    agent any

    parameters {
        choice(
            name: 'PRODUCT',
            choices: ['Agoda', 'Booking', 'Expedia', 'MakeMyTrip', 'Goibibo'],
            description: 'Select product to deploy'
        )
        choice(
            name: 'DLL_FILE',
            choices: ['FastBooking1.dll', 'GoogleHotelFinder2.dll'],
            description: 'Select DLL to deploy'
        )
    }

    environment {
        UPLOAD_DIR = 'D:\\Uploads'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/Bot-Deployment.git'
            }
        }

        stage('Validate DLL') {
            steps {
                script {
                    def dllPath = "${env.WORKSPACE}\\DLLs\\${params.DLL_FILE}"
                    if (!fileExists(dllPath)) {
                        error "❌ DLL not found in repository: ${dllPath}"
                    }
                    echo "✅ DLL found: ${dllPath}"
                    env.UPLOADED_DLL = dllPath
                }
            }
        }

        stage('Save DLL') {
            steps {
                script {
                    def targetDir = "${env.UPLOAD_DIR}"
                    def targetFile = "${targetDir}\\${params.DLL_FILE}"

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
                input message: "Confirm the DLL is correct: ${params.DLL_FILE}", ok: 'Proceed'
            }
        }

        stage('Deploy DLL') {
            steps {
                script {
                    // Example: Deployment to server
                    def servers = [
                        Agoda   : [ip:'10.10.10.54', cred:'agoda-creds', dest:'C:\\RankBot\\Agoda']
                        // Add others...
                    ]
                    def cfg = servers[params.PRODUCT]

                    withCredentials([usernamePassword(credentialsId: cfg.cred,
                        usernameVariable: 'DEPLOY_USER', passwordVariable: 'DEPLOY_PASS')]) {

                        powershell """
                        \$sec = ConvertTo-SecureString \$env:DEPLOY_PASS -AsPlainText -Force
                        \$cred = New-Object System.Management.Automation.PSCredential(\$env:DEPLOY_USER, \$sec)
                        \$s = New-PSSession -ComputerName ${cfg.ip} -Credential \$cred
                        Copy-Item '${env.UPLOAD_DIR}\\${params.DLL_FILE}' '${cfg.dest}' -ToSession \$s -Force
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
