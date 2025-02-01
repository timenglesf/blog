---
date: '2025-02-01T11:04:04-08:00'
draft: false 
title: 'Understanding systemd Targets'
---
## Introduction

Linux machines need to boot up many processes in a specific order to start the
machine in the graphical interface. This process is managed by `systemd`, the
system and service manager for many Linux distributions.

![Login screen for the graphical systemd target](/img/systemd_graphical_login.png)

## What are `systemd` targets?

`systemd` determine swhich processes to start in which order by following the
directions within in systemd target files. When an Ubuntu machine boots up it
starts executing `default.target` systemd target file which is a symbolic link
to a specific target file then executes the instructions within it.

## Checking the default `systemd` target

The default systemd target file for the system can be viewed by running the
following command:

```bash
systemctl get-default
```

When I run this command on my Ubuntu machine it outputs `graphical.target`, which
is the systemd target file with instructions on how to load the graphical
interface.

## Changing the default `systemd` target

It is possible to change the default systemd target using the following command,
which will create a symbolic link between the `default.target` file and the
`systemd_target` file.

```bash
sudo systemctl set-default <systemd_target>
```

For instance if I ran that command and used the `multi-user.target` system target,
I will have changed the default target of my machine to a text based interface.
When I next reboot my system I will no longer be taken to the graphical login
page, but a text based one instead.

There will be brief descriptions on the different systemd target at the end of
this post.

![Login screen for the multi user systemd target](/img/systemd_multi-user_login.png)

## Switching `systemd` targets

After rebooting my system with the `multi-user.target` set as my default target
I am able to revert back to using the `graphical.target` without restarting my
system by using the `isolate` command:

```bash
sudo systemctl isolate graphical.target
```

This will change the systemd target and load me into the graphical interface,
but it will not change my default systemd target, so when I reboot I will once
again be in the text based user interface.

## Common `systemd` targets

- **`graphical.target`** - Boots into the Graphical Interface
- **`multi-user.target`** - Starts the same processes and daemons as the Graphical
  Interface, but skips loading the graphical processes.
- **`emergency.target`** - Will load as few programs as possible. It can be useful
  for debugging. The root file system will be mounted as read only.
- **`rescue.target`** - A few more programs are loaded than `emergency.target`
  and you loaded in as the root user in the shell. It is useful for creating
  DB backups while not connected to a network and fixing settings.

  ## Conclusion

Understanding systemd targets is important for managing a Linux systems.
Whether troubleshooting, optimizing boot performance, or configuring a
server environment, knowing how to check, change, and switch between targets
will lend to greater control over a machine.
