# Jenkins with Docker Compose

This guide will help you set up Jenkins, a popular automation server, using Docker Compose. The following `docker-compose.yml` file defines a Jenkins service and uses Docker to create a containerized environment for Jenkins.

## Prerequisites

To get started, you'll need:

1. **Docker**: Installed on your machine. You can find installation instructions [here](https://docs.docker.com/get-docker/).
2. **Docker Compose**: Installed on your machine. Installation instructions are available [here](https://docs.docker.com/compose/install/).

## Getting Started

Below is the Docker Compose configuration that will set up Jenkins:

```yaml
docker-compose.yml
services:
  # Jenkins server
  jenkins:
    container_name: jenkins-server
    image: jenkins/jenkins:lts
    privileged: true
    user: root
    volumes:
      - /tmp/jenkins_home:/var/jenkins_home
    ports:
      - "8888:8080"   # Expose Jenkins UI on port 8080
      - "50000:50000" # Jenkins agent communication
    environment:
      JENKINS_OPTS: "--httpPort=8080"

volumes:
  jenkins_home:
    driver: local
```

### What Does This File Do?

- **Services Section**: Defines the services that Docker Compose will run. In this case, it is **Jenkins**.

  - **jenkins**: This is the main Jenkins server configuration.
    - **container_name**: Names the container as `jenkins-server`.
    - **image**: Specifies the image to use (`jenkins/jenkins:lts`). This is the long-term support version of Jenkins.
    - **privileged**: Grants extended privileges to the Jenkins container, which might be necessary for some administrative tasks.
    - **user**: Runs Jenkins as the `root` user, giving it permission to install plugins and dependencies.
    - **volumes**: Maps the local folder `/tmp/jenkins_home` to the Jenkins container folder `/var/jenkins_home`. This folder is where Jenkins stores its configuration and jobs.
    - **ports**: Maps ports on your machine to the ports inside the Jenkins container.
      - `8888:8080`: Jenkins will be available at [http://localhost:8888](http://localhost:8888). You can customize the `8888` port to avoid conflicts with other services running on your machine by changing it to a different port number, for example, `9999:8080`.
      - `50000:50000`: Port for Jenkins agent communication.
    - **environment**: Sets environment variables for the container. `JENKINS_OPTS` is used to configure Jenkins, and here it is set to use port 8080.

- **Volumes Section**: Manages Docker volumes to persist data. `jenkins_home` is defined here to store Jenkins data between container restarts.

## Running Jenkins

To run Jenkins using Docker Compose:

1. **Create the `docker-compose.yml` file**: Copy the configuration above into a file named `docker-compose.yml`.
2. **Start Jenkins**: Run the following command in the same directory as your `docker-compose.yml` file:

   ```bash
   docker-compose up -d
   ```

   The `-d` flag runs the containers in detached mode, allowing you to use your terminal for other tasks.

3. **Access Jenkins**: Open your browser and navigate to [http://localhost:8888](http://localhost:8888). Jenkins should be accessible here.

## Stopping Jenkins

To stop Jenkins, use the following command in the directory containing your `docker-compose.yml`:

```bash
docker-compose down
```

This will stop and remove the containers, but any data in `/tmp/jenkins_home` will be preserved.

## Tips for Beginners

Unlock Jenkins: The first time you access Jenkins, it will ask for an admin password. This password is stored in /tmp/jenkins_home/secrets/initialAdminPassword. You can find it by running:

  ```bash
  docker exec jenkins-server cat /var/jenkins_home/secrets/initialAdminPassword
  ```

Alternatively, you can navigate to the mounted volume on your host machine **/tmp/jenkins_home/secrets/initialAdminPassword** to retrieve the password directly.

- **Persisting Data**: Data is saved in `/tmp/jenkins_home`. If you remove this folder, Jenkins will lose all its data.
- **Customizing Jenkins**: You can install plugins directly from the Jenkins UI to extend its functionality.

## Troubleshooting

- If Jenkins is not accessible at `http://localhost:8888`, ensure Docker is running and that there are no port conflicts. If another service is already using port 8888, you can change it to a different available port in the `docker-compose.yml` file.
- Check container logs for any errors by running:

  ```bash
  docker logs jenkins-server
  ```

## Learn More

For more information about Jenkins, visit the [Jenkins documentation](https://www.jenkins.io/doc/).
