#!/usr/bin/env groovy

properties([
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '3')),
    disableConcurrentBuilds(),
    [$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false]
])

String packer = "C:\\Packer\\packer.exe"
String packerTemplate = "centos7.json"
String vmName = "Packer_centos7"
String vcenter = "vcenter.example.com"
String content_library = "content library name"
String isoDatastorePath = "[Shared Storage 01] contentlib-guid/guid/CentOS-7-x86_64-Minimal-1908_guid.iso"

node ("packer") {
    stage ('Init') { checkout scm }

    withCredentials([usernamePassword(credentialsId: 'guid', passwordVariable: 'PASSWORD', usernameVariable: 'LOGIN')]) {
        stage ("Packer") {
		    bat """
                ${packer} build --force ^
                    -var vcenter_server=${vcenter} ^
                    -var vsphere_user=${env.LOGIN} ^
                    -var vsphere_password=${env.PASSWORD} ^
                    -var \"os_iso=${isoDatastorePath}\" ^
                    -var vm_name=${vmName} ^
                    %WORKSPACE%\\${packerTemplate}
		    """
        }
        stage ("OVF template") {
            powershell """
                $credentials = New-Object System.Management.Automation.PSCredential(${env.LOGIN}, $(${env.PASSWORD} | ConvertTo-SecureString -asPlainText -Force))
                Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false | Out-Null
                Connect-CisServer -Server $VCenter -Credential $credentials -Force | Out-Null
                Connect-VIServer -Server $VCenter -Credential $credentials | Out-Null

                Add-TemplateToLibrary -LibraryName "${content_library}" `
                    -VMname '${vmName}' `
                    -LibItemName '${vmName}' `
                    -Description 'Builded automatically'
            """
        }
    }
}