# Ansible - Server Logging and Monitoring Playbook

## Infra

- ELK stack is used for log retention, analysis, filtering of different services, such as HAProxy, nginx access logs, logs produced by apps.
- Grafana, Prometheus, Node Exporter, Alert Manager is used for node monitoring, graphing metrics over the time, producing alerts based on rules defined.

The instances which needs to be monitored must have the following agents installed.

[filebeat](https://www.elastic.co/products/beats/filebeat) - To ship logs to Logstash running on control instance.

[node_exporter](https://github.com/prometheus/node_exporter) - To produce system metrics of the node instance.

Filebeat works on a `PUSH` mechanism, in a way that it reads the input files and sends the data over TCP connection to Logstash(output can be configured, but the control instances are configured to accept input from Logstash, parse and process to Elasticsearch.)

Node Exporter works on a `PULL` mechanism, in a way that it exposes an HTTP endpoint, for other services to scrape metrics data from this endpoint. Prometheus should be configured to scrape data from all instances running `node_exporter`.

## Setup

- Clone the repo to your host machine

- Create a virtualenv to install Ansible

    `python3 -m venv venv`

    `source venv/bin/activate`

    `pip install ansible`

    Make sure Ansible is installed correctly and available in your PATH by running `ansible -v`

- Create an inventory file in your working directory, which is to be used with `ansible-playbook -i <inventory_file>` command.
Refer to the sample format of an inventory file listing the hosts and extra options:

    ```
    [vagrant]
    127.0.0.1 ansible_user=vagrant ansible_ssh_port=2222 ansible_ssh_extra_args="-o IdentitiesOnly=yes"

    [prod]
    13.1.2.3 ansible_user=ubuntu ansible_ssh_port=2200
    ```
    You can read more about inventories [here](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html).


## Running the Playbook

`ansible-playbook <playbook_file>.yml -i <inventory_file> -v`

- If you want to setup a server which installs ELK stack along with Grafana, Prometheus, AlertManager, Node Exporter, run the `control.yml` playbook.
- If you want to setup Filebeat and Node Exporter in the node instances, run `node.yml` playbook.


## Notes
- If you're on `macOS` chances are that you'll need to install `GNU Tar`, the `bsd` tar isn't compatible with Ansible `unarchive` command.
Use `brew install gnu-tar` to install.

- If you're reverse proxying through a web server like `nginx`, you might be running different applications on the same server(control instance), but using `subpaths`. This is a very common scenario but took us some time to figure it out, so putting the steps here if you'd like to do the same:

    Scneario: If you have an application, (say Kibana) running on `localhost:5601` and reverse proxied to port `5000` using `nginx`, you want to proxy `grafana` running on `localhost:3000` on `localhost:5000/grafana`, you want to additionally proxy `prometheus` running on `localhost:9090` on `localhost:5000/prometheus`, you can refer to the below nginx config

    ```

        server {
            listen 5000;

                location / {
                    proxy_pass http://localhost:5601;
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection 'upgrade';
                    proxy_set_header Host $host;
                    proxy_cache_bypass $http_upgrade;
                }

            location /prometheus/ {
                proxy_pass http://localhost:9090/prometheus/;
            }

            location /grafana/ {
                proxy_pass http://localhost:3000/;
            }
        }

    ```


    Additionally you need to do the following steps:

    - Open `grafana.ini` and change the settings in `[server.domain]` setting. ([Reference](http://docs.grafana.org/installation/behind_proxy/))

    - Open `prometheus.service` set `--web.external-url`. ([Reference](https://www.robustperception.io/using-external-urls-and-proxies-with-prometheus))