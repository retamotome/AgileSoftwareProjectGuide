# GitLab Container Registry Quick Setup

This guide helps you enable the GitLab Container Registry with a self-signed certificate and connect Docker clients. It is written for clarity and quick reference.

## About GitLab Container Registry

The Container Registry lets you build, store, and share Docker images within your GitLab instance. This guide shows how to set it up with a custom domain and self-signed certificate.

## Before You Start

Make sure you have:
- Admin access to your GitLab server
- Docker installed on your client machines
- OpenSSL available on both server and client

---

## Setting Up the Registry on the GitLab Server

For full details, see the [official documentation](https://docs.gitlab.com/administration/packages/container_registry/#configure-container-registry-under-its-own-domain) and [self-signed certificate guide](https://docs.gitlab.com/administration/packages/container_registry/#configure-self-signed-certificates).

### Generate a Self-Signed Certificate (Optional)

```sh
cd /etc/gitlab/ssl/
openssl req -x509 -sha256 -newkey rsa:2048 -nodes -keyout <certificate-of-container-registry>.key \
	-subj "/CN=<your-container-domain.example.com>" \
	-addext "subjectAltName = DNS:<your-container-domain.example.com>" \
	-days 3650 -out <certificate-of-container-registry>.crt
```

> **Tip:** Change the domain and filenames to match your environment.  

### Update GitLab Configuration
In `/etc/gitlab/gitlab.rb`, specify the Container Registry domain and the paths to its SSL certificate and key:

```sh
registry_external_url 'https://<your-container-domain.example.com>'
registry_nginx['ssl_certificate'] = "/etc/gitlab/ssl/<certificate-of-container-registry>.crt"
registry_nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/<certificate-of-container-registry>.key"
```
---

## Setting Up the Docker Client
Add this line to your `/etc/hosts` file so your machine can find the domain:
```sh
<your-GitLab IP> <your-container-domain.example.com>
```
### Get the Registry Certificate

```sh
openssl s_client -showcerts -connect <your-container-domain.example.com>:443 < /dev/null | \
	sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ca.crt
```

### Trust the Certificate in Docker

**a. Create the Docker certs directory:**
```sh
sudo mkdir -p /etc/docker/certs.d/<your-container-domain.example.com>/
sudo mkdir -p /etc/docker/certs.d/://<your-container-domain.example.com>
```

**b. Copy the certificate:**
```sh
sudo cp ca.crt /etc/docker/certs.d/<your-container-domain.example.com>/
sudo cp ca.crt /etc/docker/certs.d/://<your-container-domain.example.com>
```

**c. Restart Docker to apply the changes:**
```sh
sudo systemctl restart docker
```

### Add Certificate to System CA Store
If not already installed, add the ca-certificates package:
```sh
sudo apt-get install -y ca-certificates
```
Copy the certificate to the system CA directory:
```sh
sudo cp ca.crt /usr/local/share/ca-certificates/
```
Update the certificate store:
```sh
sudo update-ca-certificates
```
---

## Logging In to the Container Registry

Now log in with your GitLab credentials:
```sh
docker login <your-container-domain.example.com>
```

> **Troubleshooting:**
> - If you see certificate errors, restart Docker:
>   ```sh
>   sudo systemctl restart docker
>   ```
> - Double-check the registry domain and directory path.
> - Make sure Docker can read the certificate files.

> **Note:** The default registry port is `:5000`. For HTTPS on port 443, you can omit the port in the directory name.

---

## References

- [GitLab Container Registry Documentation](https://docs.gitlab.com/ee/user/packages/container_registry/)
- [Configure Container Registry under its own domain](https://docs.gitlab.com/administration/packages/container_registry/#configure-container-registry-under-its-own-domain)
- [Configure Self-Signed Certificates](https://docs.gitlab.com/administration/packages/container_registry/#configure-self-signed-certificates)
- [Create Your Own SSL Certificate Authority for Local HTTPS Development](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/)


---

# License

![BY NC SA](../../img/Cc-by-nc-sa.png)  
GitLab Container Registry Quick Setup © 2026 by Jen Yuan Pan is licensed under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode.en).  

