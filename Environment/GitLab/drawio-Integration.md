# Draw.io Integration with GitLab

This guide provides step-by-step instructions to integrate Draw.io (diagrams.net) with your GitLab instance using Docker and Nginx. This setup allows you to self-host Draw.io and access it via your GitLab server.

---

## High-Level Architecture

![High-Level Architecture](./reference/img/architecture.drawio.svg)

### Network Request Flow

**Normal GitLab request**

```
Browser → http://your-GitLab-server/
        → GitLab Nginx
        → GitLab Web App
        → Response
```

**draw.io request**

```
Browser → http://your-GitLab-server/drawio/
        → GitLab Nginx
        → (external config: drawio.conf)
        → proxy_pass → http://127.0.0.1:18080
        → draw.io container
        → Response
```

## Deploy Draw.io with Docker Compose

**Create a directory for Draw.io:**

```bash
mkdir -p /opt/drawio
cd /opt/drawio
```

**Create the Docker Compose file:**

```bash
vim docker-compose.yml
```

Paste the following content:

```yaml
services:
    drawio:
      image: jgraph/drawio:latest
      container_name: drawio
      restart: always
      ports:
      - "127.0.0.1:18080:8080"
      environment:
      - PUBLIC_URL=https://localhost/drawio
```

**Start the Draw.io container:**

```bash
docker-compose up -d
```

**Verify the service is running:**

```bash
curl https://127.0.0.1:18080
```

You should receive an HTML response if Draw.io is running correctly.

---

## Configure Nginx for Draw.io Proxy

**Create an external Nginx config file for Draw.io:**

```bash
sudo vim /var/opt/gitlab/nginx/conf/drawio.conf
```

Add the following content:

```nginx
location /drawio/ {
    proxy_pass https://127.0.0.1:18080/;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

**Set correct permissions for the config file:**

```bash
sudo chmod 644 /var/opt/gitlab/nginx/conf/drawio.conf
```

**Tell GitLab to include the external config:**

```bash
sudo vim /etc/gitlab/gitlab.rb
```

Add or update the following line:

```ruby
nginx['custom_gitlab_server_config'] = "include /var/opt/gitlab/nginx/conf/drawio.conf;"

# Force HTTPS globally in GitLab  
nginx['redirect_http_to_https'] = true
```

---

## Apply and Test the Configuration

**Apply the new configuration and restart Nginx:**

```bash
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart nginx

# Confirm all services are running
sudo gitlab-ctl status
```

**Debug if needed:**

```bash
sudo gitlab-ctl tail nginx
```

---

## Access Draw.io

After completing the above steps, you can access Draw.io via:

```
https://your-GitLab-server/drawio/
```

## Enable Diagrams.net Integration

For detailed instructions on enabling Diagrams.net (Draw.io) integration with GitLab, refer to the official documentation: [Enable Diagrams.net Integration](https://gitlab.com/gitlab-org/gitlab/-/blob/8079e9878fb6b4fb0fd030c1f334457ddbde1386/doc/administration/integration/diagrams_net.md).

## Using Diagrams in the GitLab Wiki Editor

To learn how to integrate and use diagrams directly within the GitLab Wiki editor, visit the following resource: [Diagram in the GitLab Wiki Editor](https://www.drawio.com/blog/gitlab-wiki-integration).

---

# License

![BY NC ND](../../img/Cc-by-nc-sa.png)  
Draw.io Integration with GitLab © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  
