# Postgres as a rootless container in Fedora IoT

Objective is to setup and run postgres in a rootless container. Starting and stopping the container should be handled by 
a systemd service.

## Creating user
Start by create a new system user for postgres. Since we need the user's session to be running after boot, enable linger
for the user. `loginctl enable-linger postgres`

We need user-mode networking for unprivileged namespaces. [slirp4netns](https://github.com/rootless-containers/slirp4netns)
provides this functionality. It comes pre-installed in Fedora IoT.

> slirp4netns allows connecting a network namespace to the Internet in a completely unprivileged way, by connecting a 
TAP device in a network namespace to the usermode TCP/IP stack ([slirp](https://gitlab.freedesktop.org/slirp/libslirp)).

Increase the maximum number of user namespaces in Kernel
```sh
# echo “user.max_user_namespaces=28633” > /etc/sysctl.d/userns.conf 	 
# sysctl -p /etc/sysctl.d/userns.conf
```
Ansible offers `ansible.posix.sysctl` module to do this.

We also need subuid and subgid ranges set in `/etc/{subuid,setgid}` respectively. `useradd` seems to do this automatically
for regular accounts, but not for system accounts. So, we have to update the files manually. Make sure the ranges don't
overlap with ranges of existing accounts.

## Setting up Postgres
Start by *creating* a container with Podman. Don't run the container yet. Instead, generate a systemd unit file for 
the container which we will use to start and stop postgres. Create folder in `/home/postgres` and bind mount it to 
`/var/lib/postgresql/data` inside the container.

Once the container is created, generate a systemd file (`podman generate systemd`). Ansible's `containers.podman.podman_container`
module offers this functionality as well. To run the container as `postgres` user, place the systemd unit file in 
`/home/postgres/.config/systemd/user/`. Enable the service in systemd and reboot.

Don't forget to open postgres' port (TCP 5432) in firewall to allow remote connections.

## References
1. https://developers.redhat.com/blog/2020/09/25/rootless-containers-with-podman-the-basics
2. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html-single/managing_containers/index#set_up_for_rootless_containers


tags: #containers, #podman
