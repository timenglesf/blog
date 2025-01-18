---
date: '2024-08-22T17:44:14-08:00'
draft: false 
title: 'Continuous  Deployment: Creating a GitHub Workflow, Build Script, and Dockerfile'
---


*The references to a blog in this post are for my older blog that was built from
scratch using Go. My new blog is built using Hugo and the Papermod.*

In the [previous post](https://timengle.dev/posts/dynamic-file-serving-with-embedded-file-system-and-object-storage-in-go-file-serving-with-embedded-file-system-and-object-storage-in-go/), the importance of setting up continuous deployment (CD) was introduced, and a basic GitHub Actions workflow was created to automate code deployments. In this section of the series, we’ll build on that foundation by discussing three essential files needed for fully automating the deployment process. The first is a more advanced GitHub Actions workflow that will handle all the steps required to deploy updated containers to GCP. The second is a build script that will create a binary of the Go application compatible with the operating system running inside the Docker container. Finally, we’ll cover the Dockerfile, which is responsible for building the container image that will be deployed and hosted by Google Cloud Platform.

## GitHub Actions Workflow

To fully automate the CD pipeline, a service is needed to handle the entire process. In this series, GitHub Actions will be the tool of choice. The GitHub Actions Workflow will follow these steps to deploy new containers in the cloud, making them accessible to users:

1. Checkout the code
2. Setup Go
3. Build a Binary of the Go application
4. Authorize Google Cloud Platform
5. Set up & verify the Google Cloud SDK
6. Build and push the Docker container to Google Cloud
7. Deploy the newly built container

The initial GitHub Workflow below covers steps 1 and 2. Additional steps will be added throughout the series:

```yaml
on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest

    steps:
      # Checkout the repository code
      - name: Checkout code
        uses: actions/checkout@v4

      # Set up Go environment
      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: "1.22.1"

```

In simple terms, whenever code is pushed to the `main` branch of the repository, this workflow will execute on an instance running the latest version of Ubuntu, performing steps 1 and 2 outlined above.

---

## Build Script

To ensure the Go binary is compatible with the Docker container’s OS and architecture, the build process on GitHub Actions needs to match the container’s environment. The build will be done on Ubuntu, the default operating system provided by GitHub Actions. To control this, a build script called `buildprod.sh` will be placed in a `scripts` directory. This script will generate a binary that meets the container's requirements.

```bash
#!/bin/bash

# Set environment variables for cross-compilation
export GOOS=linux
export GOARCH=amd64

# Build the Go application
go build -o timengledev_blog ./cmd/web
```

Before adding a step to execute this script within the GitHub Actions workflow, ensure the script is executable by running the following command from the root of the repository:

```shell
chmod +x scripts/buildprod.sh
```

### Adding a build step to the workflow

Now that an executable script is in place to build the Go application into a binary, a step needs to be added to execute the script in the workflow.

```yaml
on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest

    steps:
      # Checkout the repository code
      - name: Checkout code
        uses: actions/checkout@v4

      # Set up Go environment
      - name: Setup Go
        uses: actions/setup-go@v5
        with:
          go-version: "1.22.1"

      # Build the production binary
      - name: Build Binary
        run: ./scripts/buildprod.sh
```

---

## Creating a Dockerfile

The Dockerfile is what defines how the container image for the Go application is built. This file provides the instructions Docker needs to package the Go binary, along with any dependencies, into a container that can be easily deployed and managed in Google Cloud Platform. Below is the Dockerfile used for this project:

```dockerfile
# Use a specific version of Debian to ensure consistency
FROM --platform=linux/amd64 debian:stable-slim

RUN apt-get update && apt-get install -y ca-certificates

# Copy the blog binary into the container's working directory
COPY ./timengledev_blog ./timengledev_blog

# Specify the entrypoint to run the blog binary
CMD ["./timengledev_blog"]

```

### Breaking down the Dockerfile

1. **Base Image Selection**: The `FROM` instruction specifies the base image. In this case, `debian:stable-slim` is used to keep the container lightweight and stable. The `--platform=linux/amd64` ensures the image runs consistently across different environments, matching the architecture set during the build script.
2. **Installing Dependencies**: The `RUN` command updates the package lists and installs `ca-certificates`, which is often needed for HTTPS connections. Keeping dependencies minimal helps reduce the overall image size.
3. **Copying the Binary**: The `COPY` instruction moves the `timengledev_blog` binary generated by the build script into the container’s working directory. This is the executable that the container will run.
4. **Setting the Entrypoint**: Finally, the `CMD` specifies the entrypoint for the container, telling Docker to run the `timengledev_blog` binary when the container starts.

---

## Conclusion

With the GitHub Actions workflow, build script, and Dockerfile in place, the foundation for our continuous deployment pipeline is set. These files ensure that every time a change is pushed to the `main` branch, the Go application is built and packaged into a Docker container that’s ready for deployment.

In the [next part](https://timengle.dev/posts/view/Continuous+Deployment%3A+Automating+GCP+Deployments+with+Artifact+Registry+and+Cloud+Run) of this series, the focus will shift to configuring Google Cloud Platform. We’ll start by setting up and managing our Docker containers in the cloud using [Artifact Registry](https://cloud.google.com/artifact-registry). From there, we’ll explore [Cloud Run](https://cloud.google.com/run) to deploy our containers with autoscaling capabilities, ensuring a seamless and scalable deployment process.

Stay tuned as we continue to build out this CD pipeline and take the next steps toward fully automating the deployment of our Go-powered blog.
