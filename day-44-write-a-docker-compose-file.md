# Day 44: Write a Docker Compose File

The Nautilus application development team shared static website content that needs to be hosted on the `httpd` web server using a containerised platform. The team has shared details with the DevOps team, and we need to set up an environment according to those guidelines. Below are the details:

a. On `App Server 3` in `Stratos DC` create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (please use the exact name for file).

b. Use `httpd` (preferably `latest` tag) image for container and make sure container is named as `httpd`; you can use any name for service.

c. Map `80` number port of container with port `8082` of docker host.

d. Map container's `/usr/local/apache2/htdocs` volume with `/opt/security` volume of docker host which is already there. (please do not modify any data within these locations).

## Specific Requirements:

1. On `App Server 3` in `Stratos DC` create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (please use the exact name for file).
2. Use `httpd` (preferably `latest` tag) image for container and make sure container is named as `httpd`; you can use any name for service.
3. Map `80` number port of container with port `8086` of docker host.
4. Map container's `/usr/local/apache2/htdocs` volume with `/opt/itadmin` volume of docker host which is already there. (please do not modify any data within these locations).

## Solution

Docker Compose defines a multi-container application's configuration in YAML, even when the application contains only one container. The compose file creates the requested Apache HTTP Server container, publishes the required port, and bind-mounts the pre-existing static website content without changing it.

### 🔐 Step 1: Connect to App Server 3

```bash
ssh banner@stapp03
```

> **Why:** `ssh` opens a remote shell on App Server 3 as `banner`, the user authorized to manage the Docker environment for this lab.

### 📝 Step 2: Create the Compose file

```bash
docker --version
docker compose version
ls -la /opt/itadmin
cd /opt/docker
sudo vi docker-compose.yml
```

Add the following content, then save and exit with `Esc`, `:wq`, and `Enter`:

```yaml
services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "8086:80"
    volumes:
      - /opt/itadmin:/usr/local/apache2/htdocs:ro
```

> **Why:** `cd /opt/docker` changes to the directory required by the challenge. `sudo vi docker-compose.yml` creates or edits the exact Compose filename with elevated permissions. `services` contains the Compose-managed containers; `web` is an arbitrary service name. `image: httpd:latest` selects the requested Apache HTTP Server image and its `latest` tag. `container_name: httpd` assigns the exact required container name. Under `ports`, `"8082:80"` forwards host port `8082` to port `80` inside the container. Under `volumes`, `/opt/itadmin` is bind-mounted at Apache's document root, `/usr/local/apache2/htdocs`. The `:ro` suffix makes this mount read-only inside the container, protecting the supplied content from application writes.

### 🚀 Step 3: Create and start the Compose service

```bash
sudo cat docker-compose.yml
sudo docker compose up -d
```

Compose pulled `httpd:latest`, created its default network, and created the `httpd` container.

```
[+] up 9/9
 ✔ Image httpd:latest     Pulled                                                                                           3.9s
 ✔ Network docker_default Created                                                                                          0.1s
 ✔ Container httpd        Created      
```

> **Why:** `docker compose up` creates the resources declared in `docker-compose.yml` and starts the service. Docker pulls `httpd:latest` automatically when it is not already present. `-d` runs the service in detached mode, keeping the container running after the command returns. `sudo` provides the permissions needed to communicate with the Docker daemon on this host.

### ✅ Step 4: Verify the container and port mapping

```bash
docker ps
```

The running container showed the expected configuration:

```text
CONTAINER ID   IMAGE          COMMAND              CREATED          STATUS          PORTS                                   NAMES
fa6d70217639   httpd:latest   "httpd-foreground"   49 seconds ago   Up 47 seconds   0.0.0.0:8086->80/tcp, :::8086->80/tcp   httpd
```

> **Why:** `docker ps` lists running containers. `Up` confirms that the `httpd` container remains running. `0.0.0.0:8082->80/tcp` confirms the requested IPv4 host-to-container port mapping, while `:::8082->80/tcp` shows the equivalent IPv6 binding.

### ✅ Step 5: Check container logs

```bash
docker logs httpd

```
Purpose: View Apache HTTPD startup logs to ensure service started correctly.
Expected Output:

AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Thu Sep 17 16:00:40.333182 2026] [mpm_event:notice] [pid 1:tid 1] AH00489: Apache/2.4.68 (Unix) configured -- resuming normal operations
[Thu Sep 17 16:00:40.333299 2026] [core:notice] [pid 1:tid 1] AH00094: Command line: 'httpd -D FOREGROUND'
172.17.0.1 - - [17/Sep/2026:16:02:55 +0000] "GET / HTTP/1.1" 200 244

### ✅ Step 6: Verify volume mount

```bash
docker exec httpd ls -la /usr/local/apache2/htdocs
```
Purpose: Confirm that host directory is properly mounted inside the container.

Expected Result: Should show same files as /opt/itadmin on host.

total 16
drwxr-xr-x 2 root     root     4096 Sep 17 15:50 .
drwxr-xr-x 1 www-data www-data 4096 Aug 25 00:22 ..
-rw-r--r-- 1 root     root       34 Sep 17 15:50 index1.html

### ✅ Step 7: Test website from host

```bash
curl http://localhost:8084

```
Purpose: Test Apache HTTPD is serving static content on the mapped port.

Expected Output: HTML content from the mounted static website files.

## Best Practices

- **Use declarative container definitions.** Keeping image, port, and volume settings in `docker-compose.yml` makes the deployment repeatable and easier to review.
- **Protect supplied static content.** Mounting `/opt/security` with `:ro` lets Apache serve the files while preventing container processes from writing to them.
- **Publish only required ports.** Exposing `8082` only for the HTTP service limits unnecessary network exposure.
- **Verify the effective configuration.** Inspect `docker ps` after startup to confirm both the container state and the actual published port.

### 📚 Official Documentation

- [Docker Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Docker Compose `up` reference](https://docs.docker.com/reference/cli/docker/compose/up/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Publishing and mapping ports](https://docs.docker.com/get-started/docker-concepts/running-containers/publishing-ports/)
