# About

This is a Docker on Windows Server 2022 (21H2) Vagrant environment for playing with Windows containers.

For Windows Server 2019 (1809) see the [rgl/docker-windows-2019-vagrant](https://github.com/rgl/docker-windows-2019-vagrant) repository.

# Usage

Install the [Base Windows Server 2022 Box](https://github.com/rgl/windows-vagrant).

Install the required plugins:

```bash
vagrant plugin install vagrant-reload
```

Then launch the environment:

```bash
vagrant up --no-destroy-on-error --provider=libvirt # or --provider=virtualbox
```

At the end of the provision the [examples](examples/) are run.

The Docker Engine API endpoint is available at http://10.0.0.3:2375.

# Graceful Container Shutdown

**Windows containers cannot be gracefully shutdown** because they are forcefully terminated after a while. Check the [moby issue 25982](https://github.com/moby/moby/issues/25982) for progress.

The next table describes whether a `docker stop --time 600 <container>` will graceful shutdown a container that is running a [console](https://github.com/rgl/graceful-terminating-console-application-windows/), [gui](https://github.com/rgl/graceful-terminating-gui-application-windows/), or [service](https://github.com/rgl/graceful-terminating-windows-service/) app.

| base image                                    | app     | behavior                                                                                     |
| --------------------------------------------- | ------- | -------------------------------------------------------------------------------------------- |
| mcr.microsoft.com/windows/nanoserver:ltsc2022 | console | receives the `CTRL_SHUTDOWN_EVENT` notification but is killed after about 5 seconds          |
| mcr.microsoft.com/windows/servercore:ltsc2022 | console | receives the `CTRL_SHUTDOWN_EVENT` notification but is killed after about 5 seconds          |
| mcr.microsoft.com/windows/server:ltsc2022     | console | receives the `CTRL_SHUTDOWN_EVENT` notification but is killed after about 5 seconds          |
| mcr.microsoft.com/windows/nanoserver:ltsc2022 | service | receives the `SERVICE_CONTROL_PRESHUTDOWN` notification but is killed after about 15 seconds |
| mcr.microsoft.com/windows/servercore:ltsc2022 | service | receives the `SERVICE_CONTROL_PRESHUTDOWN` notification but is killed after about 15 seconds |
| mcr.microsoft.com/windows/server:ltsc2022     | service | receives the `SERVICE_CONTROL_PRESHUTDOWN` notification but is killed after about 15 seconds |
| mcr.microsoft.com/windows/nanoserver:ltsc2022 | gui     | fails to run because there is no GUI support libraries in the base image                     |
| mcr.microsoft.com/windows/servercore:ltsc2022 | gui     | does not receive the shutdown messages `WM_QUERYENDSESSION` or `WM_CLOSE`                    |
| mcr.microsoft.com/windows/server:ltsc2022     | gui     | does not receive the shutdown messages `WM_QUERYENDSESSION` or `WM_CLOSE`                    |

**NG** setting `WaitToKillServiceTimeout` (e.g. `Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control -Name WaitToKillServiceTimeout -Value '450000'`) does not have any effect on extending the kill service timeout.

**NB** setting `WaitToKillAppTimeout` (e.g. `New-ItemProperty -Force -Path 'HKU:\.DEFAULT\Control Panel\Desktop' -Name WaitToKillAppTimeout -Value '450000' -PropertyType String`) does not have any effect on extending the kill application timeout.

You can launch these example containers from host as:

```bash
vagrant execute --sudo -c '/vagrant/ps.ps1 examples/graceful-terminating-console-application/run.ps1'
vagrant execute --sudo -c '/vagrant/ps.ps1 examples/graceful-terminating-windows-service/run.ps1'
vagrant execute --sudo -c '/vagrant/ps.ps1 examples/graceful-terminating-gui-application/run.ps1'
```

# Docker images

This environment builds and uses the following images:

```
REPOSITORY                            TAG                        IMAGE ID      CREATED         SIZE
busybox-info                          latest                     84ff098d976e  35 minutes ago  296MB
go-info                               latest                     29d72267de9a  35 minutes ago  297MB
csharp-info                           latest                     4b56a5fdc60e  35 minutes ago  369MB
powershell-info                       latest                     c09e433ffab3  37 minutes ago  569MB
batch-info                            latest                     938a556b3d0f  37 minutes ago  295MB
busybox                               latest                     491a8bf48c87  37 minutes ago  296MB
golang                                1.17.3                     343e41f8f94b  37 minutes ago  779MB
mcr.microsoft.com/powershell          7.2.0-nanoserver-ltsc2022  1ab8ce50a0cb  4 days ago      566MB
mcr.microsoft.com/dotnet/sdk          6.0-nanoserver-ltsc2022    c48ce22cdf23  11 days ago     1GB
mcr.microsoft.com/dotnet/runtime      6.0-nanoserver-ltsc2022    adfdff5cbe9f  11 days ago     368MB
mcr.microsoft.com/windows/server      ltsc2022                   06f23dce1e0c  2 weeks ago     11.3GB
mcr.microsoft.com/windows/servercore  ltsc2022                   3d4792f88e26  2 weeks ago     4.91GB
mcr.microsoft.com/windows/nanoserver  ltsc2022                   8c7c323428d9  2 weeks ago     295MB
```

# Troubleshoot

* Restart the docker daemon in debug mode and watch the logs:
  * set `"debug": true` inside the `$env:ProgramData\docker\config\daemon.json` file
  * restart docker with `Restart-Service docker`
  * watch the logs with `Get-EventLog -LogName Application -Source docker -Newest 50`
* For more information see the [Microsoft Troubleshooting guide](https://docs.microsoft.com/en-us/virtualization/windowscontainers/troubleshooting) and the [CleanupContainerHostNetworking](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools/CleanupContainerHostNetworking) page.

# References

* [Using Insider Container Images](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/using-insider-container-images)
* [Beyond \ - the path to Windows and Linux parity in Docker (DockerCon 17)](https://www.youtube.com/watch?v=4ZY_4OeyJsw)
* [The Internals Behind Bringing Docker & Containers to Windows (DockerCon 16)](https://www.youtube.com/watch?v=85nCF5S8Qok)
* [Introducing the Host Compute Service](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/)
