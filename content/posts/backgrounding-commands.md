---
date: '2025-02-05T19:13:55-08:00'
draft: false 
title: 'Backgrounding Commands'
---

When working in Linux's terminal you do not always want to wait for a command to
finish executing before continuing to work on the command line. In order to
overcome this block background jobs can be used. In order to have a command run
in the background, the `&` symbol can be appended to the command and you will
arrive back at your prompt while the command runs in the background. This post
will be a quick guide on how to start, manage, and move jobs from the background
and foreground.

### Running a Command in the Background

Below I have a command that will run for 5 minutes, and will be placing it into
the background by employing the `&` symbol. Since the command is instructed to
run in the background, I will immediately be placed back into my prompt ready
to continue working.

```bash
sleep 300 &
```

### Viewing Background Jobs

To check which jobs are running in the background the `jobs` command can be used:

```bash
tim@tim-thinkpad:~$ jobs
[1]+  Stopped                 nvim
[2]-  Running                 sleep 300 &
```

### Bringing a Background Job to the Foreground

If it is necessary to bring a background job back into the foreground the `fg`, foreground, command along with the job number can be used.

```bash
fg 2
```

Now the `sleep 300` command should be placed back into the foreground.

### Moving a Foreground Job to the Background

 To place the command back into the background again I will follow these steps:

1. Stop the command by using `CTRL-Z`, which will place me back into the terminal
prompt, BUT will stop the job from running. Check the status of the job again
with the `jobs` command, it should read as `stopped`.
2. In order to have the job continue in the background I can use the `bg`, b
ackground command, along with the job number like so:

```bash
bg 2
```

Now the job is running in the background again, and it is possible to continue working while waiting for the background job to complete.

### Conclusion

Managing background jobs in the terminal is a simple way to keep a workflow
smooth while in the terminal. By making use of the `&`, `jobs`, `fg`, and `bg`
command processes can be controlled easily without losing progress or having to
start over. These commands can be especially useful for long running processes
like backups.
