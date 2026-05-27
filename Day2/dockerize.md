***

# Day 02: Dockerizing Your First Application 🐳

Welcome to Day 2 of the Certified Kubernetes Administrator (CKA) series! In this guide, we cover the practical, hands-on steps to dockerize a sample Node.js application from scratch. You will learn how to write a `Dockerfile`, build an image, push it to Docker Hub, and run your container.

---

## 🛠️ Prerequisites & Environment Setup

Before writing any code, you need a Docker environment. Choose the option that best fits your current hardware setup:

| Setup Option | Best For | Instructions |
| :--- | :--- | :--- |
| **Docker Desktop** | Machines with good resources (Mac, Windows, Linux) | Download from [docker.com](https://www.docker.com/products/docker-desktop) and follow the installation wizard. |
| **Play with Docker** | Low-resource machines or quick, temporary testing | Go to [labs.play-with-docker.com](https://labs.play-with-docker.com), log in with Docker Hub credentials, and start a 4-hour sandbox session. |

> **Note:** We will be using the local Docker Desktop setup for the commands below, but they will work exactly the same in the Play with Docker terminal.

---

## Step 1: Get the Application Source Code

We need a sample application to containerize. We will use a standard To-Do List app provided by Docker.

1. Create a new directory for your project and navigate into it:
```bash
   mkdir day02-code
   cd day02-code
```

2. Clone the sample repository:
   
```bash
   git clone https://github.com/docker/getting-started-app.git
```

3. Navigate into the cloned application folder:
```bash
cd getting-started-app
```

---

## Step 2: Write the Dockerfile

A `Dockerfile` is a text document that contains all the commands needed to assemble an image.

1. Create an empty Dockerfile:
```bash
touch Dockerfile
```

2. Open the file in a text editor (like `vi`, `nano`, or VS Code) and add the following instructions:
```dockerfile
   FROM node:18-alpine
   WORKDIR /app
   COPY . .
   RUN yarn install --production
   CMD [\"node\", \"src/index.js\"]
   EXPOSE 3000
```

**What do these commands mean?**
* **`FROM node:18-alpine`**: Sets the base operating system. We use Alpine Linux because it is incredibly lightweight, and this specific version already has Node.js installed.
* **`WORKDIR /app`**: Creates an `/app` directory inside the container and sets it as the default location for all subsequent commands.
* **`COPY . .`**: Copies all files from your local machine (first `.`) to the current working directory inside the container (second `.`).
* **`RUN yarn install --production`**: Installs only the necessary application dependencies.
* **`CMD [\"node\", \"src/index.js\"]`**: The default command that executes when the container starts.
* **`EXPOSE 3000`**: Documents that the container will listen on port 3000.

---

## Step 3: Build the Docker Image

Now we tell the Docker engine to read our `Dockerfile` and build the image layer by layer.

1. Run the build command (don't forget the dot at the end, which specifies the current directory):
```bash
docker build -t day02-todo .
```

2. Verify that your image was created successfully:
```bash
docker images
```
   *You should see `day02-todo` in the list with the tag `latest`.*

---

## Step 4: Push the Image to Docker Hub

To share your application or deploy it to other environments (like Dev, Test, or Prod), you need to store it in a central registry like Docker Hub.

1. Log in to Docker Hub via the terminal:
```bash
docker login
```
   *Enter your Docker Hub username and password when prompted.*

2. Tag your local image with your Docker Hub username and repository name:
```bash
docker tag day02-todo:latest <your-username>/test-repo:latest
```

3. Push the image to the remote registry:
```bash
docker push <your-username>/test-repo:latest
```

---

## Step 5: Run the Container

Your image is built and stored. Now, let's bring the application to life by running a container based on that image.

1. Execute the `docker run` command:
```bash
docker run -d -p 3000:3000 <your-username>/test-repo:latest
```
   * **`-d`**: Runs the container in \"detached\" mode (in the background).
   * **`-p 3000:3000`**: Maps port 3000 on your local machine to port 3000 inside the container.

2. Check if the container is running:
```bash
docker ps
```

3. Open your web browser and go to `http://localhost:3000`. You should see your To-Do application running!

---

## Step 6: Troubleshooting & Executing into a Container

If your application isn't working as expected, you might need to look inside the running container.

1. Find your container's Name or ID using `docker ps`.
2. Open an interactive shell inside the container:
```bash
docker exec -it <container-name-or-id> sh
```
   *We use `sh` instead of `bash` because the lightweight Alpine base image does not include bash by default.*

---

## 🚀 Next Steps & Challenge

If you run `docker images`, you might notice that our resulting image size is around 200MB+. For a simple static app on Alpine Linux, that is significantly larger than it needs to be! 

**Challenge:** How can we optimize this `Dockerfile` to reduce the image size and implement better security/build practices? 

