# escape=`
ARG BUILD_IMAGE
ARG BASE_IMAGE

FROM $BUILD_IMAGE as build

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# download remote assets
RUN New-Item -Path 'C:\\downloads' -ItemType 'Directory' -Force | Out-Null; `
    & curl.exe -sS -L -o c:\\downloads\\nuget.exe https://dist.nuget.org/win-x86-commandline/v5.2.0/nuget.exe; `
    & curl.exe -sS -L -o C:\\downloads\\urlrewrite.msi https://download.microsoft.com/download/1/2/8/128E2E22-C1B9-44A4-BE2A-5859ED1D4592/rewrite_amd64_en-US.msi; `
    & curl.exe -sS -L -o C:\\downloads\\vc_redist.exe https://aka.ms/vs/15/release/VC_redist.x64.exe; `
    & curl.exe -sS -L -o C:\\downloads\\filebeat.zip https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.1-windows-x86_64.zip; `
    & curl.exe -sS -L -o C:\\downloads\\dotnet-hosting.exe https://download.visualstudio.microsoft.com/download/pr/34f4b2a6-c3b8-495c-a11f-6db955f27757/8c340c1a8c25966e39e0c0a4b308dff4/dotnet-hosting-2.2.5-win.exe; `
    & curl.exe -sS -L -o c:\\downloads\\node.msi https://nodejs.org/dist/v12.13.0/node-v12.13.0-x64.msi;

# copy local assets
COPY *.zip C:\\downloads\\
COPY patches\\ c:\\patches\\

# install nuget
RUN New-Item -Path 'C:\\install\\tools\\bin' -ItemType 'Directory' -Force | Out-Null; `
    Move-Item -Path 'C:\\downloads\\nuget.exe' -Destination 'C:\\install\\tools\\bin' -Force;

# install microsoft xdt assembly
RUN & 'C:\\install\\tools\\bin\\nuget.exe' install 'Microsoft.Web.Xdt' -Version '3.0.0' -OutputDirectory 'C:\\install'; `
    Copy-Item -Path 'C:\\install\\Microsoft.Web.Xdt*\\lib\\netstandard2.0\\*.dll' -Destination 'C:\\install\\tools\\bin'; `
    Remove-Item -Path (Get-Item -Path 'C:\\install\\Microsoft.Web.Xdt*\\').FullName -Recurse -Force;

# install serilog assembly
RUN & 'C:\\install\\tools\\bin\\nuget.exe' install 'Serilog.Sinks.UDP' -Version '6.0.0' -OutputDirectory 'C:\\install\\serilog'; `
    New-Item -Path 'c:\\install\\bin\\serilog' -ItemType 'Directory' -Force | Out-Null; `
    Copy-Item -Path 'C:\\install\\serilog\\Serilog.Sinks.PeriodicBatching*\\lib\\netstandard2.0\\*.dll' -Destination 'C:\\install\\bin\\serilog\\'; `
    Copy-Item -Path 'C:\\install\\serilog\\Serilog.Sinks.Udp*\\lib\\netstandard2.0\\*.dll' -Destination 'C:\\install\\bin\\serilog\\'; `
    Remove-Item -Path (Get-Item -Path 'C:\\install\\Serilog\\').FullName -Recurse -Force;

# install filebeat
RUN Expand-Archive -Path 'C:\\downloads\\filebeat.zip' -DestinationPath 'C:\\install\\tools\\bin' -Force; `
    Rename-Item -Path (Get-Item -Path 'C:\\install\\tools\\bin\\filebeat*windows*').FullName -NewName "filebeat"; `
    Remove-Item -Path 'C:\\downloads\\filebeat.zip' -Force;

# extract assets, skip wdps, move already extracted wdps
RUN $zips = Get-ChildItem -Path 'C:\\downloads\\*.zip' -Exclude '*.scwdp.zip'; `
    $zips | ForEach-Object { Expand-Archive -Path $_.FullName -DestinationPath 'C:\\packages' -Force; }; `
    $zips | ForEach-Object { Remove-Item -Path $_.FullName -Force; }; `
    $zips = Get-ChildItem -Path 'C:\\downloads\\*.zip' -Exclude '*Configuration files*.zip'; `
    $zips | ForEach-Object { Move-Item -Path $_.FullName -Destination 'C:\\packages'; };

# move installers
RUN New-Item -Path 'C:\\install\\setup' -ItemType 'Directory' -Force | Out-Null; `
    Get-ChildItem 'C:\\downloads\\*.*' -Include '*.exe', '*.msi' | Move-Item -Destination 'C:\\install\\setup';

# add tools folder
COPY tools\\ C:\\install\\tools\\

FROM $BASE_IMAGE

COPY --from=build ["C:\\install\\", "C:\\install\\"]
COPY --from=build ["C:\\packages\\", "C:\\packages\\"]
COPY --from=build ["C:\\patches\\", "C:\\patches\\"]
