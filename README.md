Authentication/Encryption Setup for Prometheus with Node Exporter
This guide details setting up authentication and encryption for Prometheus alongside Node Exporter using self-signed certificates and basic authentication.

Important Note:

This guide uses self-signed certificates, which are not recommended for production environments due to security concerns. Consider using a trusted Certificate Authority (CA) for a more secure setup.
1. Certificate Generation for Node Exporter:

Generate self-signed certificates:

Bash
sudo openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -subj "/C=US/ST=California/L=Oakland/O=MyOrg/CN=localhost" -addext "subjectAltName = DNS:localhost"
Use code with caution.
content_copy
This command generates a key (node_exporter.key) and a certificate (node_exporter.crt) with the specified details.
Create Node Exporter directory:

Bash
sudo mkdir /etc/node_exporter
Use code with caution.
content_copy
Move certificates:

Bash
sudo mv node_exporter.* /etc/node_exporter/
Use code with caution.
content_copy
Configure TLS for Node Exporter:

Create a file named config.yml inside the directory (/etc/node_exporter/) with the following content:

YAML
tls_server_config:
  cert_file: node_exporter.crt
  key_file: node_exporter.key
Use code with caution.
content_copy
Set permissions:

Bash
sudo chown -R node_exporter:node_exporter /etc/node_exporter
Use code with caution.
content_copy
This grants ownership of the directory and its contents to the node_exporter user.
Update systemd service:

Edit the Node Exporter systemd service file (/etc/systemd/system/node_exporter.service) to include the TLS configuration (using vi editor):

Bash
sudo vi /etc/systemd/system/node_exporter.service
Use code with caution.
content_copy
Update the service configuration to include the following under the [Service] section:

ExecStart=/usr/local/bin/node_exporter --web.config.file="/etc/node_exporter/config.yml"
Save and close the file.

Reload systemd and restart Node Exporter:

Bash
sudo systemctl daemon-reload
sudo systemctl restart node_exporter
Use code with caution.
content_copy
2. Update Prometheus Configuration:

Transfer certificate:

Securely copy the node_exporter.crt certificate to the Prometheus server's /etc/prometheus directory using a method like scp (refer to previous steps for secure transfer methods).
Bash
# Replace with your username and server IP
sudo scp /etc/node_exporter/node_exporter.crt aya@192.168.56.110:/etc/prometheus
Use code with caution.
content_copy
Set permissions:

Bash
sudo chown prometheus:prometheus /etc/prometheus/node_exporter.crt
Use code with caution.
content_copy
Update Prometheus configuration (prometheus.yml):

Edit your Prometheus configuration file (prometheus.yml) and update the scrape configuration for the Node Exporter with the following changes:

Set scheme to https
Add basic_auth section with username and password (replace with your desired credentials)
Add tls_config section with ca_file pointing to the copied certificate and set insecure_skip_verify to true (**caution: this is for demonstration purposes only, avoid using it in production).
YAML
scrape_configs:
- job_name: 'Linux Server'
  scheme: https
  basic_auth:
    username: prometheus
    password: aya1
  tls_config:
    ca_file: "/etc/prometheus/node_exporter.crt"
    insecure_skip_verify: true  # Not recommended for production
  static_configs:
  - targets: ['192.168.56.111:9100']
Use code with caution.
content_copy
Restart Prometheus:

Bash
sudo systemctl restart prometheus
Use code with caution.
