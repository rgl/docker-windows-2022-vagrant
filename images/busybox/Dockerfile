# escape=`
ARG POWERSHELL_IMAGE
ARG WINDOWS_NANOSERVER_IMAGE

FROM ${POWERSHELL_IMAGE} AS builder
SHELL ["pwsh.exe", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]
RUN cd $env:TMP; `
    $url = 'https://frippery.org/files/busybox/busybox-w64-FRP-4487-gd239d2d52.exe'; `
    $sha256 = '23d1bf00d07cf090abf1360eea2233195aeb5a99fcd35bf3ad1e833c76c00c6b'; `
    Write-Host ('Downloading {0}...' -f $url); `
    Invoke-WebRequest -Uri $url -OutFile busybox.exe; `
    Write-Host ('Verifying sha256 ({0})...' -f $sha256); `
    if ((Get-FileHash busybox.exe -Algorithm sha256).Hash -ne $sha256) { `
        Write-Host 'FAILED!'; `
        Exit 1; `
    };

FROM ${WINDOWS_NANOSERVER_IMAGE}
COPY --from=builder /Windows/TEMP/busybox.exe /Windows/System32/
