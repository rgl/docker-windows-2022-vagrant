# escape=`
ARG POWERSHELL_IMAGE
ARG WINDOWS_NANOSERVER_IMAGE

FROM ${POWERSHELL_IMAGE} AS builder
SHELL ["pwsh.exe", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]
RUN cd c:/Windows/Temp; `
    $url = 'https://frippery.org/files/busybox/busybox-w64-FRP-5467-g9376eebd8.exe'; `
    $sha256 = '9d04939789a8d458ea25ffe8fdd788ae4edc15408936da0e5c1394f02cb21a43'; `
    Write-Host ('Downloading {0}...' -f $url); `
    Invoke-WebRequest -Uri $url -OutFile busybox.exe; `
    Write-Host ('Verifying sha256 ({0})...' -f $sha256); `
    if ((Get-FileHash busybox.exe -Algorithm sha256).Hash -ne $sha256) { `
        Write-Host 'FAILED!'; `
        Exit 1; `
    };

FROM ${WINDOWS_NANOSERVER_IMAGE}
COPY --from=builder /Windows/Temp/busybox.exe /Windows/System32/
