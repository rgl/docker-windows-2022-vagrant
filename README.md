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
vagrant up --provider=virtualbox # or --provider=libvirt
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
busybox-info                          latest                     9e1d2543886c  37 minutes ago  298MB
go-info                               latest                     ab2a7261c83d  37 minutes ago  299MB
csharp-info                           latest                     50407fe512c8  37 minutes ago  369MB
powershell-info                       latest                     3cc16e447387  39 minutes ago  557MB
batch-info                            latest                     e8a65d0195a3  40 minutes ago  297MB
busybox                               latest                     556cf106dfe2  40 minutes ago  298MB
golang                                1.17.0                     db9e2e805138  40 minutes ago  783MB
mcr.microsoft.com/powershell          7.1.4-nanoserver-ltsc2022  61ded2895024  4 days ago      553MB
mcr.microsoft.com/dotnet/sdk          6.0-nanoserver-ltsc2022    ba0220de3c63  5 days ago      1.01GB
mcr.microsoft.com/dotnet/runtime      6.0-nanoserver-ltsc2022    323199b4ca0c  5 days ago      369MB
mcr.microsoft.com/windows/servercore  ltsc2022                   801c33f5de3a  2 weeks ago     5.1GB
mcr.microsoft.com/windows/server      ltsc2022                   d86596fba4fc  2 weeks ago     12.3GB
mcr.microsoft.com/windows/nanoserver  ltsc2022                   9126f7df26a0  2 weeks ago     297MB
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
