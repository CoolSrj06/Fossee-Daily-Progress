*Pre-requisite: Docker and Docker-compose must be installed*

First, you must clone the [openproject-docker-compose](https://github.com/opf/openproject-docker-compose) repository:

```bash
git clone https://github.com/opf/openproject-docker-compose.git --depth=1 --branch=stable/17 openproject
```

Copy the example `.env` file and edit any values you want to change:

```bash
cp .env.example .env
vim .env
```

If you are using the default value of OPDATA that is used in the `.env.example` you need to make sure that the folder exist, and you have the right permissions:

```bash
sudo mkdir -p /var/openproject/assets
sudo chown 1000:1000 -R /var/openproject/assets
```

Next you start up the containers in the background while making sure to pull the latest versions of all used images.

```bash
OPENPROJECT_HTTPS=false docker compose up -d --build --pull always
```

After a while, OpenProject should be up and running on `http://localhost:8080`
The default username and password is login: `admin`, and password: `admin`.

Reference: [OpenProject installation with Docker Compose - OpenProject](https://www.openproject.org/docs/installation-and-operations/installation/docker-compose/)

![alt text](<../images/Pasted image 20260119125518.png>)

# Zulip Integration Steps

1. [Create a bot](https://zulip.fossee.in/help/add-a-bot-or-integration) for OpenProject. Make sure that you select **Incoming webhook** as the **Bot type**.
2. Decide where to send OpenProject notifications, and [generate the integration URL](https://zulip.fossee.in/help/generate-integration-url).
3. From your OpenProject organization, click on your user profile icon. Select **Administration** from the dropdown menu, and navigate to **API and Webhooks**. Select the **Webhooks** tab from the left panel, and click on **+ Webhook**.
4. Enter a name of your choice for the webhook, such as `Zulip`. Set **Payload URL** to the URL generated above, and ensure the webhook is enabled.
5. Select the events and projects you want to receive notifications for, and click **Create**.

## Upon doing this your webhook might not work, you  need to check your webhook OpenProject Webhook Logs

OpenProject records whether its outgoing webhook requests succeeded or failed.

1. **Log in to OpenProject** as an Admin.
2. Go to **Administration** -> **API and Webhooks** -> **Webhooks**.
3. Click on the specific webhook you created for Zulip.
4. Look for a **Recent Deliveries** or **Logs** section (depending on your version).
    - **Click on a failed delivery:** It will show you the exact response body Zulip sent back. This is often more useful than the server logs because it will contain the user-facing error message (e.g., "Invalid stream name").

**CLI Method (Docker):** If the UI doesn't show details, check the Docker logs for the OpenProject worker process (which handles background webhooks):

Bash

```bash
# Adjust container name (often 'openproject-worker-1' or similar)
docker logs --tail 100 -f openproject_container_name
```

*If you get error like: Failed to open TCP connection to zulip.servertest.org:8443 (getaddrinfo(3): Name or service not known)*

This error confirms that the problem is **DNS Resolution** within the Docker container.

1. Open your OpenProject `docker-compose.yml` file
2. Find the `web` and/or `worker` services (OpenProject has both, webhooks are often processed by the worker).
3. Add the `extra_hosts` section:
	```yml
	worker:
    <<: *app
    command: "./docker/prod/worker"
    networks:
      - backend
    depends_on:
      - db
      - cache
      - seeder
    extra_hosts:
      - "zulip.servertest.org:192.168.56.101"
    volumes:
      - ./zulip.crt:/usr/local/share/ca-certificates/zulip.crt:ro
	```

Replace `HOST_IP_ADDRESS` with the actual LAN IP of your Podman host (e.g., `192.168.1.50`). Do not use `127.0.0.1`.

## Next You Might Get SSL Error

*SSL_connect returned=1 errno=0 peeraddr=192.168.56.101:8443 state=error: certificate verify failed (self-signed certificate)*

This error means your DNS fix worked (OpenProject found the server), but now it refuses to talk to it because it doesn't trust the self-signed certificate you created for Zulip.

You need to take the certificate from the Zulip container and install it into the OpenProject container's "trusted store."

#### Step 1: Get the Certificate from Zulip

Run this command on your host machine to copy the certificate out of the Zulip container:

```bash
# Copy the cert from the zulip container to your current directory
# Adjust '/etc/ssl/certs/zulip-selfsigned.crt' to where your actual cert lives in the container
podman cp zulip:/etc/ssl/certs/zulip-selfsigned.crt ./zulip.crt
```

_(If you created the cert manually on your host, just use that file.)_

#### Step 2: Mount it into OpenProject

You need to map this file into the OpenProject container's special certificate directory: `/usr/local/share/ca-certificates/`.

**If using Docker Compose:** Add a volume mapping to your `openproject` service:

```yaml
services:
  openproject:
    volumes:
      - ./zulip.crt:/usr/local/share/ca-certificates/zulip.crt:ro
```

**If using `docker run`:** Add this flag:

```bash
-v $(pwd)/zulip.crt:/usr/local/share/ca-certificates/zulip.crt:ro
```

#### Step 3: Update the Trust Store

Just mounting the file isn't enough; you have to tell the OS to read it.

1. Restart the OpenProject container so the file is mounted.
2. Run this command **inside** the OpenProject container:
```bash
docker exec -u root -it openproject_container_name update-ca-certificates
```
    (You should see output like: `1 added, 0 removed; done.`)_
3. **Restart the OpenProject application** (or the whole container one last time) to ensure the Ruby worker process picks up the new certificate store.

