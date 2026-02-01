*Pre-requisite: Docker and Docker-compose must be already installed on you vm*

### Phase 1: Install Kanboard (with Docker Compose)

Using `docker-compose` is better than a single command because it saves your data (persistence) and makes it easier to update later.

1. **Create a folder** for your project:

```bash
mkdir kanboard
cd kanboard
```

2. **Create a `docker-compose.yml` file** inside that folder:

```yaml
services:
  kanboard:
    image: kanboard/kanboard:latest
    container_name: kanboard
    ports:
      - "8080:80"  # You can change 8080 to any port you like
    volumes:
      - kanboard_data:/var/www/app/data
      - kanboard_plugins:/var/www/app/plugins
    environment:
      - PLUGIN_INSTALLER=true  # Critical: Enables plugin installation via UI
      - DEBUG=true
      - LOG_DRIVER=stderr
    extra_hosts:
      - "zulip.servertest.org:192.168.56.101"
    restart: unless-stopped

volumes:
  kanboard_data:
  kanboard_plugins:
```

3. **Start the container**:

```bash
docker-compose up -d
```

4. **Access Kanboard**:

- Open your browser and go to `http://localhost:8080`.
- **Default Login:** `admin`
- **Default Password:** `admin` (Change this immediately in "My Profile").

4. Command to read KanBoard logs 
```bash
docker logs -f kanboard
```

---

### Phase 2: Get the Webhook URL from Zulip

We will use Zulip's "Slack Compatibility" mode. This tricks Kanboard into thinking it is talking to Slack, which is natively supported.

1. Open **Zulip**.
2. Go to **Personal Settings** (gear icon) -> **Bots**.
3. Click **Add a new bot**.
    - **Bot type:** Select `Incoming webhook`.
    - **Name:** Give it a name (e.g., "Kanboard").
    - **Stream:** Choose the stream where notifications should go (e.g., `#project-updates`).
4. Click **Create bot**.
5. **Copy the URL** provided.
    - **Crucial Step:** Look at the URL. It ends with `/api/v1/external/zulip_incoming_webhook`.
    - **Change the end** to `/api/v1/external/slack`.
    - _Example:_
        - **Original:** `https://zulip.example.com/api/v1/external/zulip_incoming_webhook?api_key=...`
        - **New:** `https://zulip.example.com/api/v1/external/slack?api_key=...`
		![alt text](<../images/Pasted image 20260119140942.png>)

6. You will almost certainly get the `certificate verify failed` error again because Zulip uses a self-signed certificate.

	Kanboard does not have a UI setting to disable SSL checks; you must do it in the config file. Since you are using Docker, run this command to create a config file that disables SSL verification:

```bash
docker exec kanboard sh -c "echo \"<?php define('HTTP_VERIFY_SSL_CERTIFICATE', false);\" > /var/www/app/data/config.php"
```
---

### Phase 3: Connect Kanboard to Zulip

Now you need to tell Kanboard to send data to that URL.

1. **Install the Slack Plugin**:
    - In Kanboard, click the **dropdown menu** (top right) -> **Plugins**.
    - Click **Directory**.
    - Search for **Slack**.
    - Click **Install**.
    - _Note: If you don't see an "Install" button, ensure you added `PLUGIN_INSTALLER=true` in your docker-compose file and restarted the container._

2. **Configure the Project**:
    - Go to your specific **Project** (e.g., "My Task Board").
    - Click **Settings** (left sidebar) -> **Integrations**.
    - Scroll down to the **Slack** section.
    - **Webhook URL:** Paste the _modified_ Zulip URL you created in Phase 2.
    - **Channel:** Enter the stream name (e.g., `#project-updates`).
    - Click **Save**.

### Phase 4: Test the Connection

1. Go to your Kanboard board.
2. **Create a new task** or move an existing task to a different column.
3. Check your **Zulip** stream. You should see a notification instantly.
![alt text](<../images/Pasted image 20260119142520.png>)
![alt text](<../images/Pasted image 20260119142615.png>)