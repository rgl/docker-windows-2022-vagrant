# escape=`
ARG WINDOWS_NANOSERVER_IMAGE
ARG POWERSHELL_IMAGE

FROM ${POWERSHELL_IMAGE} AS builder
SHELL ["pwsh.exe", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]
WORKDIR C:/build
RUN function Install-Artifact($url, $hash, $to) { `
        Write-Host ('Downloading {0}...' -f $url); `
        Invoke-WebRequest -Uri $url -OutFile artifact.zip; `
        Write-Host ('Verifying if the hash is {0}...' -f $hash); `
        if ((Get-FileHash artifact.zip -Algorithm sha256).Hash -ne $hash) { `
            Write-Host 'FAILED!'; `
            Exit 1; `
        }; `
        Write-Host 'Expanding...'; `
        mkdir -Force $to | Out-Null; `
        Expand-Archive artifact.zip $to; `
        Write-Host 'Removing...'; `
        Remove-Item artifact.zip -Force; `
        Write-Host 'Done.'; `
    }; `
    Install-Artifact `
        'https://golang.org/dl/go1.17.3.windows-amd64.zip' `
        'e78684b955742e215926204afc6ed62b9d165b509e25a687d62902516f08726b' `
        '.'; `
    Install-Artifact `
        'https://github.com/git-for-windows/git/releases/download/v2.34.0.windows.1/MinGit-2.34.0-64-bit.zip' `
        'e53818f68308a8f7988f37e2b56f4e8da2aa93b279b6a70c87a21972c7c4ab93' `
        'git';
RUN ./git/cmd/git config --file git/etc/gitconfig core.autocrlf false

FROM ${WINDOWS_NANOSERVER_IMAGE}
COPY --from=builder C:/build/go C:/go
COPY --from=builder C:/build/git C:/git
ENV PATH='C:\Windows\System32;C:\Windows;C:\Program Files\PowerShell;C:\go\bin;C:\git\cmd'
