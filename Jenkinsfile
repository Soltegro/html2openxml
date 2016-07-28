#!groovy

def getWorkspace() {
    pwd().replace("%2F", "_")
}

node('Windows&&Office2016') {
    ws(getWorkspace()) {
        properties([[
                $class: 'BuildDiscarderProperty',
                strategy: [$class: 'LogRotator', artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '25']
            ]])

        stage 'Checkout'
        checkout scm

        stage 'Restore Dependencies'
        bat ".NuGet\\NuGet.exe restore HtmlToOpenXml.sln"

        stage 'Build'
        bat "\"${tool name: 'MSBuild 14', type: 'hudson.plugins.msbuild.MsBuildInstallation'}\" HtmlToOpenXml.sln /p:configuration=Debug"
        bat "\"${tool name: 'MSBuild 14', type: 'hudson.plugins.msbuild.MsBuildInstallation'}\" HtmlToOpenXml.sln /p:configuration=Release"

        stage 'CodeAnalysis'
        bat "\"${tool name: 'MSBuild 14', type: 'hudson.plugins.msbuild.MsBuildInstallation'}\" HtmlToOpenXml.sln /p:RunCodeAnalysis=true"

        stage 'Archive'
        archive 'Trunk/bin/**/*'

        stage 'Publish Reports'
        step([
                $class: 'WarningsPublisher',
                canComputeNew: false,
                canResolveRelativePaths: false,
                consoleParsers: [[parserName: 'MSBuild']],
                defaultEncoding: '',
                excludePattern: '',
                healthy: '',
                includePattern: '',
                messagesPattern: '',
                unHealthy: ''
            ])

        step([
                $class: 'Mailer',
                notifyEveryUnstableBuild: false,
                recipients: '',
                sendToIndividuals: true])
    }
}