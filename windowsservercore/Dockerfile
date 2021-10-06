FROM mcr.microsoft.com/dotnet/framework/sdk:4.8-windowsservercore-2004

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

COPY scripts/*.cs /scripts/

# https://github.com/corretto/corretto-11/releases
ARG jdkWindowsComponent='https://corretto.aws/downloads/resources/11.0.12.7.1/amazon-corretto-11.0.12.7.1-windows-x64-jdk.zip'
ARG jdkWindowsComponentMD5SUM='022bf679bc9c337287859264496c2f38'

# https://dotnet.microsoft.com/download/dotnet/6.0
ARG dotnetWindowsComponent='https://download.visualstudio.microsoft.com/download/pr/c50149b6-4e1f-4278-869a-742aa2c7cfe5/bc936112cb6f6215b456c6c1e77d8826/dotnet-sdk-6.0.100-rc.1.21463.6-win-x64.zip'
ARG dotnetWindowsComponentSHA512='96d807e79baa7d69d9248736ea9a63ddcc53281a63765b47dcfd9ded24038cc445834647ef375ffad17ea83aee6f95d234109825f885353ccca73f6684a5acd0'

RUN [Net.ServicePointManager]::SecurityProtocol = 'tls12, tls11, tls' ; \
    $code = Get-Content -Path "scripts/Web.cs" -Raw ; \
    Add-Type -TypeDefinition "$code" -Language CSharp ; \
    $downloadScript = [Scripts.Web]::DownloadFiles($Env:jdkWindowsComponent + '#MD5#' + $Env:jdkWindowsComponentMD5SUM, 'jdk.zip', $Env:dotnetWindowsComponent + '#SHA512#' + $Env:dotnetWindowsComponentSHA512, 'dotnet.zip') ; \
    Remove-Item -Force -Recurse $Env:ProgramFiles\dotnet; \
    Expand-Archive dotnet.zip -Force -DestinationPath $Env:ProgramFiles\dotnet; \
    Remove-Item -Force dotnet.zip; \
    Get-ChildItem -Path $Env:ProgramFiles\dotnet -Include *.lzma -File -Recurse | foreach { $_.Delete()}; \
    Expand-Archive jdk.zip -DestinationPath $Env:ProgramFiles\Java ; \
    Get-ChildItem $Env:ProgramFiles\Java | Rename-Item -NewName "OpenJDK" ; \
    Remove-Item $Env:ProgramFiles\Java\OpenJDK\lib\src.zip -Force ; \
    Remove-Item -Force jdk.zip ; \
    Remove-Item -Recurse -Force /scripts ;

    # Java home directory
ENV JAVA_HOME="C:\Program Files\Java\OpenJDK" \
    # Opt out of the telemetry feature
    DOTNET_CLI_TELEMETRY_OPTOUT=true \
    # Disable first time experience
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=true \
    # Configure Kestrel web server to bind to port 80 when present
    ASPNETCORE_URLS=http://+:80 \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps perfomance
    NUGET_XMLDOC_MODE=skip

USER ContainerAdministrator
RUN setx /M PATH ('{0};{1}\bin' -f $env:PATH, $env:JAVA_HOME)
USER ContainerUser
