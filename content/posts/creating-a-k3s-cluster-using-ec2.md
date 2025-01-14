---
date: '2025-01-13T08:51:24-08:00'
draft: false 
title: 'Creating a K3s Cluster Using EC2'
---
As part of my preparation for the GitHub Actions certification, I’ve been
itching to get some hands-on experience deploying to a Kubernetes cluster.
I have one requirement, I do not need the cluster running all the time.
Leaving it idle would just rack up unnecessary costs. My solution?
A cloud computer I can use as a controlplane node which I can easily spin up
and down as needed.

After doing some research, I decided to go with k3s. It’s lightweight,
simple to use, and perfectly suited for my needs. For hosting, I chose
AWS EC2 because I’m already familiar with it, and it offers a straightforward
experience.

---

## Setting up the EC2 Instance

First, I set up an EC2 instance with an Elastic IP to make connecting to the
cluster easier. Initially, I tried using a t2.micro instance, but it quickly
became clear that 1GB of memory wasn’t cutting it for the control plane
components. So, I upgraded to a t2.small instance, which comes with 2GB of
memory. Problem solved!

---

## Troubleshooting Connectivity Issues

After installing k3s, I ran into my first hiccup: I couldn’t connect to the
cluster from my GitHub Actions workflow. A quick dive into the logs revealed
the culprit, my Elastic IP wasn’t included in the TLS certificate.

To fix this, I reinstalled k3s and included the `--tls-san` flag with my Elastic
IP. Here’s the updated install command:

```bash
curl -sfL https://get.k3s.io | sh -s - server \
  --tls-san <my-elastic-ip>
```

Once I made this change, my workflow was able to access the cluster!

---

## Starting and Stopping the Cluster on the fly

Since I do not need the server running 24/7 I need a simple way of starting and
stopping the server without having to deal with the hassle of navigating the AWS
web console. My solution was to create a script which I could run to start and
stop the server directly from my terminal.

```bash
#!/bin/bash
# k3s-ec2-manager.sh

AWS_INSTANCE_ID="<my-instance-id>"
AWS_REGION="<aws-region>"

case $1 in
"start")
  aws ec2 start-instances --instance-ids $AWS_INSTANCE_ID --region $AWS_REGION
  echo "Starting instance..."
  ;;
"stop")
  aws ec2 stop-instances --instance-ids $AWS_INSTANCE_ID --region $AWS_REGION
  echo "Stopping instance..."
  ;;
*)
  echo "Usage: $0 {start|stop}"
  exit 1
  ;;
esac
```

---

## Conclusion

Now that I have a script to manage my EC2 instance, I can easily start and stop
a kubernetes cluster whenever I need it. This way, I can keep my costs down
while still getting the hands-on experience studying deployment methods in
preperation for the GitHub Actions certification.

In order to save more costs, in the near future I will be setting up a workflow
which uses terraform to create a spot instance which will be used to host the
k3s cluster as I am not concerned with the uptime of the cluster.
