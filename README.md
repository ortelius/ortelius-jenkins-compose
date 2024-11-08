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

## Tips

Unlock Jenkins: The first time you access Jenkins, it will ask for an admin password. This password is stored in /tmp/jenkins_home/secrets/initialAdminPassword. You can find it by running:

  ```bash
  docker exec jenkins-server cat /var/jenkins_home/secrets/initialAdminPassword
  ```

Alternatively, you can navigate to the mounted volume on your host machine `/tmp/jenkins_home/secrets/initialAdminPassword` to retrieve the password directly.

- **Persisting Data**: Data is saved in `/tmp/jenkins_home`. If you remove this folder, Jenkins will lose all its data.
- **Customizing Jenkins**: You can install plugins directly from the Jenkins UI to extend its functionality.

## Troubleshooting

- If Jenkins is not accessible at `http://localhost:8888`, ensure Docker is running and that there are no port conflicts. If another service is already using port 8888, you can change it to a different available port in the `docker-compose.yml` file.
- Check container logs for any errors by running `docker ps | grep jenkins`
- Use the container ID to view the logs

```shell
docker ps | grep jenkins

CONTAINER ID   IMAGE                                        COMMAND                  CREATED         STATUS                      PORTS                                                                         NAMES
76bc66b817f1   jenkins/jenkins:lts                          "/usr/bin/tini -- /u…"   2 minutes ago   Up 2 minutes                0.0.0.0:50000->50000/tcp, 0.0.0.0:8888->8080/tcp                              jenkins-server
```

```shell
docker logs 76bc66b817f1 # replace with your container ID
```

```shell
docker logs 76bc66b817f1
Running from: /usr/share/jenkins/jenkins.war
webroot: /var/jenkins_home/war
2024-11-07 16:59:36.807+0000 [id=1]	INFO	winstone.Logger#logInternal: Beginning extraction from war file
2024-11-07 16:59:39.021+0000 [id=1]	WARNING	o.e.j.ee9.nested.ContextHandler#setContextPath: Empty contextPath
2024-11-07 16:59:39.050+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: jetty-12.0.13; built: 2024-09-03T03:04:05.240Z; git: 816018a420329c1cacd4116799cda8c8c60a57cd; jvm 17.0.13+11
2024-11-07 16:59:39.378+0000 [id=1]	INFO	o.e.j.e.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.ee9.jsp.JettyJspServlet
2024-11-07 16:59:39.430+0000 [id=1]	INFO	o.e.j.s.DefaultSessionIdManager#doStart: Session workerName=node0
2024-11-07 16:59:39.838+0000 [id=1]	INFO	hudson.WebAppMain#contextInitialized: Jenkins home directory: /var/jenkins_home found at: EnvVars.masterEnvVars.get("JENKINS_HOME")
2024-11-07 16:59:39.950+0000 [id=1]	INFO	o.e.j.s.handler.ContextHandler#doStart: Started oeje9n.ContextHandler$CoreContextHandler@19650aa6{Jenkins v2.479.1,/,b=file:///var/jenkins_home/war/,a=AVAILABLE,h=oeje9n.ContextHandler$CoreContextHandler$CoreToNestedHandler@3ce53f6a{STARTED}}
2024-11-07 16:59:39.957+0000 [id=1]	INFO	o.e.j.server.AbstractConnector#doStart: Started ServerConnector@282ffbf5{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
2024-11-07 16:59:39.963+0000 [id=1]	INFO	org.eclipse.jetty.server.Server#doStart: Started oejs.Server@726e5805{STARTING}[12.0.13,sto=0] @3542ms
2024-11-07 16:59:39.963+0000 [id=26]	INFO	winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
2024-11-07 16:59:40.141+0000 [id=33]	INFO	jenkins.InitReactorRunner$1#onAttained: Started initialization
2024-11-07 16:59:40.175+0000 [id=54]	INFO	jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2024-11-07 16:59:40.762+0000 [id=32]	INFO	jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2024-11-07 16:59:40.766+0000 [id=32]	INFO	jenkins.InitReactorRunner$1#onAttained: Started all plugins
2024-11-07 16:59:40.766+0000 [id=49]	INFO	jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2024-11-07 16:59:40.996+0000 [id=35]	INFO	jenkins.InitReactorRunner$1#onAttained: System config loaded
2024-11-07 16:59:40.997+0000 [id=35]	INFO	jenkins.InitReactorRunner$1#onAttained: System config adapted
2024-11-07 16:59:40.997+0000 [id=42]	INFO	jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2024-11-07 16:59:40.998+0000 [id=50]	INFO	jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2024-11-07 16:59:41.025+0000 [id=49]	INFO	jenkins.install.SetupWizard#init:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

866bd1dd34e744d1b0bde1caff6181bd # Your password

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2024-11-07 16:59:46.214+0000 [id=51]	INFO	jenkins.InitReactorRunner$1#onAttained: Completed initialization
2024-11-07 16:59:46.263+0000 [id=25]	INFO	hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
```


## Learn More

For more information about Jenkins, visit the [Jenkins documentation](https://www.jenkins.io/doc/).
