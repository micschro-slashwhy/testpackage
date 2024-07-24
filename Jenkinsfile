pipeline {
    environment {
        DEVOPS_CLIENT_ID = credentials('DevopsClientId')
        DEVOPS_CLIENT_SECRET = credentials('DevopsClientSecret')
        DEVOPS_TENANT_ID = credentials('DevopsTenantId')
        DEVOPS_SCOPE = credentials('DevopsScope')
        DEVOPS_FEED_URL = credentials('DevOpsFeedUrl')
    }
    
    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:8.0'
            label 'docker'
        }
    }
    
    stages {
        stage('Build application') {
            steps {
			    bat 'dotnet pack -c Release'
            }
        }
        
        stage('Push NuGet package to Azure Devops') {
             steps {
                    // Install azure cli
                    pwsh '''
                        $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile ./AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item ./AzureCLI.msi
                    '''
                    // Login
                    pwsh 'az login --service-principal --allow-no-subscriptions --username %DEVOPS_CLIENT_ID% --password %DEVOPS_CLIENT_SECRET% --tenant %DEVOPS_TENANT_ID%'
                    // Retrieve access token and save in nuget config
                    pwsh '''
                        $response = az account get-access-token --scope "%DEVOPS_SCOPE%/.default"
                        $key = 'accessToken'
                        $json = $response | ConvertTo-Json
                        Write-Host $json.$key

                        $config = @"<?xml version="1.0" encoding="utf-8"?>
                                <configuration>
                                  <packageSources>
                                      <add key="AzureDevOps" value="%DEVOPS_FEED_URL%" />
                                  </packageSources>
                                  <packageSourceCredentials>
                                    <AzureDevOps>
                                      <add key="Username" value="PAT" />
                                      <add key="ClearTextPassword" value="{0}"/>
                                    </AzureDevOps>
                                  </packageSourceCredentials>
                                </configuration>" -f $json.$key

                    $config | Out-File -FilePath nuget.config
                    '''
                    // Push nuget
                    pwsh 'dotnet nuget push --source "AzureDevOps" --api-key az bin/Release/TestPackage.4.2.0.nupkg'
                    //pwsh 'nuget push ./%API_VERSION%/nugets/*.nupkg -ApiKey AzureDevOps -Source AzureDevOps -ConfigFile ./nuget.config'
                }
        }
    }
}