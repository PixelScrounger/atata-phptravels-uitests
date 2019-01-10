properties([
    parameters([
        string (name: 'branchName', defaultValue: 'master', description: 'Branch to get the tests from')
    ])
])

def nunitStash = []
def isFailed = false
def buildArtifactsFolder = "C:/BuildPackagesFromPipeline/$BUILD_ID"
def branch = params.branchName
currentBuild.description = "Branch: $branch"

node('master')
{
    stage('Checkout')
    {
        git branch: branch, url: 'https://github.com/PixelScrounger/atata-phptravels-uitests.git'
    }

    stage('Restore NuGet')
    {
        powershell '.\\build.ps1 RestorePackages'
    }

    stage('Build Solution')
    {
        powershell '.\\build.ps1 Build'
    }

    stage('Copy Build Artifacts')
    {
        powershell ".\\build.ps1 CopyArtifacts -BuildArtifactsFolder $buildArtifactsFolder"
    }
}

catchError
{
    isFailed = true
    stage('Tests')
    {
        parallel FirstTest: {
            node('master')
            {
                bat "C:/Dev/NUnit.Console-3.9.0/nunit3-console.exe $buildArtifactsFolder/PhpTravels.UITests.dll --where cat==FirstTest --result=TestResult1.xml"
                stash name: "FirstTestResults", includes: "TestResult1.xml"
                nunitStash += "TestResult1.xml"
            }
        }, SecondTest: {
            node('Slave1')
            {
                bat "C:/Dev/NUnit.Console-3.9.0/nunit3-console.exe $buildArtifactsFolder/PhpTravels.UITests.dll --where cat==SecondTest --result=TestResult2.xml"
                stash name: "SecondTestResults", includes: "TestResult2.xml"
                nunitStash += "TestResult2.xml"
            }
        }
    }
    isFailed = false
}

node('master')
{
    stage('Reporting')
    {
        if(nunitStash)
        {
            dir('NUnitResults')     
            {
                unstash "FirstTestResults"
                unstash "SecondTestResults"

                archiveArtifacts '*.xml'
                nunit testResultsPattern: '*.xml'
            }
        }
        
        /*
        if (!isFailed)
        {
            slackSend color: "good", message: "All tests passed.\nBranch: $branch\bBuild number: $env.BUILD_NUMBER"
        }
        else
        {
            slackSend color: "danger", message: "Tests failed.\nBranch: $branch\bBuild number: $env.BUILD_NUMBER"
        }*/
    }    
}