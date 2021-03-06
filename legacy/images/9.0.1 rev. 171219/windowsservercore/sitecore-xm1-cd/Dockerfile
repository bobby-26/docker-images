# escape=`
ARG BASE_IMAGE

FROM $BASE_IMAGE

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV SITENAME='sc'
ARG SQLADMIN_PASSWORD='HASH-epsom-sunset-cost7!'
ARG SIF_CONFIG=${INSTALL_TEMP}\\sitecore-XM1-cd.json
ARG SIF_PACKAGE=${INSTALL_TEMP}\\'Sitecore*_cd.scwdp.zip'

COPY . ${INSTALL_TEMP}

# Install Sitecore, cleanup and start site
RUN Expand-Archive -Path (Join-Path $env:INSTALL_TEMP '*.zip') -DestinationPath $env:INSTALL_TEMP; `
    Expand-Archive -Path (Join-Path $env:INSTALL_TEMP '*Configuration files*.zip') -DestinationPath $env:INSTALL_TEMP; `
    $config = Get-Content $env:SIF_CONFIG | Where-Object { $_ -notmatch '^\s*\/\/'} | Out-String | ConvertFrom-Json; `
    $config.Variables.'Site.PhysicalPath' = 'C:\inetpub\{0}' -f $env:SITENAME; `
    ConvertTo-Json $config -Depth 50 | Set-Content -Path $env:SIF_CONFIG; `
    Install-SitecoreConfiguration -Path $env:SIF_CONFIG `
    -Package $env:SIF_PACKAGE `
    -LicenseFile (Join-Path $env:INSTALL_TEMP '\\license.xml') `
    -Sitename $env:SITENAME `
    -SqlServer "sql" `
    -SqlDbPrefix $env:SITENAME `  
    -SqlCoreUser "sa" `
    -SqlCorePassword $env:SQLADMIN_PASSWORD `
    -SqlWebUser "sa" `
    -SqlWebPassword $env:SQLADMIN_PASSWORD `
    -SqlFormsUser "sa" `
    -SqlFormsPassword $env:SQLADMIN_PASSWORD `
    -SolrUrl "http://solr:8983/solr" `
    -SolrCorePrefix $env:SITENAME `
    -Skip "CreateHostHeader", "CreateBindings", "StartAppPool", "StartWebsite"; `
    Remove-Item -Path 'C:\\*.log'; `
    Remove-Item -Path ('C:\\inetpub\\{0}\\App_Data\\logs' -f $env:SITENAME) -Force -Recurse; `
    Copy-Item -Path (Join-Path $env:INSTALL_TEMP '\\Sitecore') -Destination 'C:/' -Recurse -Force; `
    $iisPath = ('IIS:\Sites\{0}' -f $env:SITENAME); `
    Set-WebConfiguration -PSPath $iisPath -Filter '/system.web/customErrors/@mode' -Value 'Off'; `
    Remove-Item -Path $env:INSTALL_TEMP -Force -Recurse; `
    Start-WebAppPool -Name $env:SITENAME; `
    Start-Website -Name $env:SITENAME;

