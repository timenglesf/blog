---
date: '2024-08-26T17:47:11-08:00'
draft: false 
title: 'Continuous Deployment: Automating GCP Deployments with Artifact Registry and Cloud Run'
---

*The references to a blog in this post are for my older blog that was built from
scratch using Go. My new blog is built using Hugo and the Papermod.*

In the [previous post](https://timengle.dev/posts/view/Continuous++Deployment%3A+Creating+a+GitHub+Workflow%2C+Build+Script%2C+and+Dockerfile), we created our GitHub Actions workflow, build script, and Dockerfile. In this post, we will focus on setting up GCP and enabling the APIs that allow our workflow to automate each step of the deployment process.

Here’s what we’ll cover:

- Setting up a GCP account and project
- Creating and configuring a service account for GitHub workflow access
- Building and pushing Docker images to Artifact Registry
- Preparing the Cloud Run API for automatic container deployment

By the end of this post we will have a fully automated pipeline which will deploy new containers to GCP whenever updates are pushed to our `main` branch of our repository.

---

## Creating a GCP Account & Project

### Creating an account

Creating a GCP account is pretty straight forward. You just need to follow the [sign up process](https://cloud.google.com/?hl=en). Starting account activates a free year long trial which you should be able to use to finish this series of posts.

### Creating a project

Once your account is created you will want to [start your first project](https://developers.google.com/workspace/guides/create-project). Give your project a name that will be easy to remember, I will name mine `timengledev`, and then we will be ready to move on to create a service account.

Before continuing, we will also need to [set up a billing account](https://cloud.google.com/billing/docs/how-to/create-billing-account) to verify ourself with GCP. Don’t worry—you should stay within the free tier if you follow the instructions in this series.

Now that our GCP account is set up we are ready to start enabling the services we need in order to store, host, and deploy Docker images and containers.

---

## Creating an Artifact Registry

We will be using Google Artifact Registry to store our Docker images. Google Artifact Registry is similar to Docker Hub, but it is private and hosted by GCP. Our goal is to create and push a new Docker image to the Artifact Registry whenever a new version of our Go application is pushed to our `main` branch.

### Enabling the Artifact Registry

1. Head to the Artifact Registry using the search input in the GCP console and enable the API.
2. Create a new repository. I used the following settings

- Name: `app`
- Format: `Docker`
- Mode: `Standard`
- Location Type: `Region`
- Region For Deployment: `us-west1`
- Left 'Google-managed encryption key' checked

Great now our Artifact Registry is set up for our project and we can continue on to create a Service Account so we can automate this build process in our GitHub Action workflow.

---

## Creating a Service Account

To gain access to our GCP project we will want to set up and create a key for a Service Account. We can follow these instructions to create a JSON key which we can use as a GitHub Secret to gain access to our GCP account in our workflow.

1. Head to the [IAM & Service Accounts](https://console.cloud.google.com/iam-admin/serviceaccounts) and select the current project you are working with.
2. At the top of the console click 'Create Service Account'.
3. Create a Service Account (I named mine `timengleblog_cloud`) and enable these permissions.

- `Cloud Build Editor`
- `Cloud Build Service Account`
- `Cloud Run Admin`
- `Service Account User`
- `Viewer`

4. Click the actions ellipsis next to the account you just created
5. Click Manage keys
6. You will be taken to a new page where you can click "ADD KEY" and select "Create new key" and select `JSON`
7. The key will download

Great we now have the credentials to begin automating our CD pipeline within our workflow.

---

## Accessing GCP Within our Workflow

We can begin to access our GCP account in our workflow by adding a new step that uses this [google-github-actions](https://github.com/google-github-actions/setup-gcloud) which will set up and configure, `gcloud` the cli tool used to access and make changes to our project via our service worker.

### Adding our JSON key

We will head to the GitHub repository for our project and navigate to
`Settings -> Secrets and Variables -> Actions`
Then copy the contents of the JSON file which contains the GCP service account key.
We will now create a new secret called `GCP_CREDENTIALS` by clicking 'New repository secret'.

Great now that we have added our credentials of our service worker we are ready to access GCP within our workflow.

### Adding a step to authenticate GCP

To allow our workflow to interact with the Google Cloud we need to authenticate the workflow using the secret we added to our repository in the previous step. This will be done using [setup-cloud action](https://github.com/google-github-actions/setup-gcloud) which will configure the `gcloud` CLI in our workflow. We will add this step following our step that runs our script responsible for building our application binary.

```yaml
# Authenticate with Google Cloud
- id: auth
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_CREDENTIALS }}

# Set up the Google Cloud SDK
- name: Setup GCP SDK
  uses: google-github-actions/setup-gcloud@v2
```

These steps will allow us to access the Artifact Registry API and begin building and pushing our Docker Images to the cloud.

---

## Building and Pushing our Updated Docker Image

We are now able to build and push our Docker images to our Artifact Registry with each push to the `main` branch of our repository. We just need to add a step that will build our Docker image that includes our updated binary. We will accomplish this by running our Dockerfile. Below is the `gcloud`command that we will use to do this.

```bash
gcloud builds submit --tag REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY/IMAGE:TAG .
```

This command builds, tags, and pushes our Docker Image. We will need to insert the correct information for the URL to push our Image to the correct Artifact Registry location. You can enter this manual or you can head to the Artifact Registry in the console, click on the name you gave your Artifact Registry and copy the url.

![Image of url clipboard icon location](https://storage.googleapis.com/timengledev-blog-bucket/static/dist/blog_img/2024_08_26/20240825090020.png)

Great, now that you have located the URL for your Artifact Registry we are ready to add the step to build and push our updated Image.

### Adding Build and Push Step

After the steps that authenticate our workflow to use `gcloud` tool we can add our step to build and push our Docker Image.

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
      
      # Authenticate with Google Cloud
      - id: auth
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_CREDENTIALS }}

      # Set up the Google Cloud SDK
      - name: Setup GCP SDK
        uses: google-github-actions/setup-gcloud@v2

      # Verify gcloud CLI setup
      - name: Use gcloud cli
        run: gcloud info

      # Build and push the Docker image to Artifact Registry
      - name: Build & Push image to Artifact Registry
        run: |
          gcloud builds submit \
            --tag us-west1-docker.pkg.dev/timengledev-blog/app/timengledev-blog:latest .
```

Now is a great time to push our updates to our GitHub repository and create a Pull Request into main. Once we accept the Pull Request our Workflow will begin running, you can view it's progress in the Actions tab of your repository. Once completed we should have a copy of our most up to date Docker Image stored within our Artifact Registry.

Great job!! We are now ready to begin setting up Cloud Run to deploy our Docker Image.

---

## Setting up Cloud Run

Now that our Docker Images are hosted on the Google Cloud using Artifact Registry we need a way to deploy, host, and scale them. The [Cloud Run API](https://cloud.google.com/run/) can do just that for us, so we need to set it up within our project so we can access it within our workflow and start deploying our most up to date version of our Docker Containers.

### Creating our Cloud Run Service

1. Head to the [Cloud Run](https://console.cloud.google.com/run) in the console.
2. Click "Create Service"
3. Select A Container Image URL

- Click "Select"
- Click on the URL that matches our Artifact Registry
- Select Latest

4. Give it a service name. I named mine `timengledev`
5. Select an appropriate region
6. Check "Allow unauthenticated invocations". This will allow anyone to access our service without having to log into GCP.
7. Open the `Container(s), Volumes, Networking, Security` tab

- Scroll down and select the Max number of instances.
- I selected `4`
- Cloud Run scales based on traffic, and if traffic grows becomes larger than our 4 containers can handle, requests will begin to be queued. You are welcome to set your max instances higher, but be aware as your traffic grows you will be charged more as the number of containers hosted at the same time grows.

8. For Ingress control select "All". This will allow direct access to our service
9. Create your service.
10. Once it has deployed you can click the link and see your service hosted!

Now that our Cloud Run service is all set up and hosting containers we can head back to our workflow and add a step which will automatically host our most up to date Images from our Artifact Registry when a push to the `main` branch occurs.

---

## Deploying our Updated Container

The final step of our workflow is to configure Cloud Run with the correct image and deployment region. We’ll use the `gcloud` CLI tool to point Cloud Run to the right project and Docker image, while also specifying settings like the `max-instances` and region for deployment.

We will need to run the following `gcloud` command in our final step to deploy our updated container using Cloud Run.

```bash
gcloud run deploy CLOUD_RUN_NAME \
 --image REGION-docker.pkg.dev/PROJECT_ID/REPOSITORY/IMAGE:TAG \
    --region REGION \
 --allow-unauthenticated \
 --project PROJECT_NAME \
 --max-instances=4
```

### Complete workflow

Now putting it all together I have a complete CD pipeline that is fully automated using a GitHub Action workflow that looks like this.

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
        
      # Authenticate with Google Cloud
      - id: auth
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_CREDENTIALS }}

      # Set up the Google Cloud SDK
      - name: Setup GCP SDK
        uses: google-github-actions/setup-gcloud@v2

      # Verify gcloud CLI setup
      - name: Use gcloud cli
        run: gcloud info

      # Build and push the Docker image to Artifact Registry
      - name: Build & Push image to Artifact Registry
        run: |
          gcloud builds submit \
            --tag us-west1-docker.pkg.dev/timengledev-blog/app/timengledev-blog:latest .

      # Deploy the container image to Cloud Run
      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy timengledev-blog \
            --image us-west1-docker.pkg.dev/timengledev-blog/app/timengledev-blog:latest \
            --region us-west1 \
            --allow-unauthenticated \
            --project timengledev-blog \
            --max-instances=4
```

When ever updates are pushed to the `main` branch the following will happen.

1. An instance of the latest ubuntu will be spun up on a remote machine.
2. The code from the main branch will be checked out into this machine.
3. Go will be set up.
4. The build script created in he previous section will execute and create our binary that runs the application.
5. The machine will be authenticated with GCP using our Service Account.
6. `gcloud` will be set up and run
7. A new Image containing the updated binary will be built and pushed to the Artifact Registry
8. Cloud Run will deploy an updated container using the updated image that was just built and pushed to the Artifact Registry.

---

## Conclusion

Now that we have a fully automated CD pipeline in place we will have more time free to focus on creating new features and fixing bugs. We will no longer need to manually build, upload, and deploy production code manually to GCP as all of this will be handled by our pipeline.

In the future we will look into monitoring containers, optimizing performance, and scaling as applications grow. This is just the beginning of creating a robust, scalable, and automated infrastructure for your Go containers.
