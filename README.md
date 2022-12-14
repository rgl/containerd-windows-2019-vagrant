# About

This is a containerd on Ubuntu and Windows Server 2019 (1809) Vagrant environment for playing with Windows containers.

For Docker on Windows Server 2019 (1809) see the [rgl/docker-windows-2019-vagrant](https://github.com/rgl/docker-windows-2019-vagrant) repository.

# Usage

Install the [Base Ubuntu 22.04 Box](https://github.com/rgl/ubuntu-vagrant).

Install the [Base Windows Server 2019 Box](https://github.com/rgl/windows-vagrant).

Install the required plugins:

```bash
vagrant plugin install vagrant-reload
# see https://github.com/hashicorp/vagrant/issues/12445#issuecomment-876566065
export CFLAGS='-I/opt/vagrant/embedded/include/ruby-3.0.0/ruby'
vagrant plugin install vagrant-hosts
```

Then launch the environment:

```bash
vagrant up --provider=libvirt --no-destroy-on-error --no-tty
```

Enter the `windows` virtual machine:

```bash
vagrant ssh windows
```

Test executing a `nanoserver` container with `ctr`:

```powershell
ctr image pull mcr.microsoft.com/windows/nanoserver:1809
ctr run --cni --rm mcr.microsoft.com/windows/nanoserver:1809 test cmd /c ver
ctr run --cni --rm mcr.microsoft.com/windows/nanoserver:1809 test cmd /c set
ctr run --cni --rm mcr.microsoft.com/windows/nanoserver:1809 test ipconfig /all
ctr run --cni --rm mcr.microsoft.com/windows/nanoserver:1809 test curl https://httpbin.org/user-agent
```

Test executing a multi-platform image container with `ctr`:

```powershell
ctr image pull docker.io/ruilopes/example-docker-buildx-go:v1.10.0
ctr run --cni --rm docker.io/ruilopes/example-docker-buildx-go:v1.10.0 test
```

Test executing a nanoserver container with `nerdctl`:

```powershell
nerdctl run --rm mcr.microsoft.com/windows/nanoserver:1809 cmd /c ver
nerdctl run --rm mcr.microsoft.com/windows/nanoserver:1809 cmd /c set
# NB nerdctl cannot yet connect to the network. see the caveats bellow.
nerdctl run --rm mcr.microsoft.com/windows/nanoserver:1809 cmd /c ipconfig /all
nerdctl run --rm mcr.microsoft.com/windows/nanoserver:1809 cmd /c curl https://httpbin.org/user-agent
```

Test executing a multi-platform image container with `nerdctl`:

```powershell
nerdctl run --rm ruilopes/example-docker-buildx-go:v1.10.0
```

# Caveats

* There is no support for networking in `nerdctl` (but there is in `ctr`).
  * See https://github.com/containerd/nerdctl/issues/559
* There is no support for building Windows containers because `buildkitd` is not available for Windows.
  * See https://github.com/moby/buildkit/issues/616
* See all the known Windows issues at https://github.com/containerd/nerdctl/labels/platform%2FWindows.

# Troubleshoot

* See the [Microsoft Troubleshooting guide](https://docs.microsoft.com/en-us/virtualization/windowscontainers/troubleshooting) and the [CleanupContainerHostNetworking](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/windows-server-container-tools/CleanupContainerHostNetworking) page.

# References

* [Tech Community Windows Server Containers News](https://techcommunity.microsoft.com/t5/containers/bg-p/Containers)
* [Using Insider Container Images](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/using-insider-container-images)
* [Beyond \ - the path to Windows and Linux parity in Docker (DockerCon 17)](https://www.youtube.com/watch?v=4ZY_4OeyJsw)
* [The Internals Behind Bringing Docker & Containers to Windows (DockerCon 16)](https://www.youtube.com/watch?v=85nCF5S8Qok)
* [Introducing the Host Compute Service](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/)
