Server Monitoring Playbook

# Infra

# Setup

# Running it on host

`ansible-playbook control.yml -i inventory -v --ssh-extra-args="-o IdentitiesOnly=yes"`

# Notes **TODO**
- If you're on `macOS` chances are that you'll need to install `GNU Tar`, the `bsd` tar isn't compatible with Ansible `unarchive` command.
Use `brew install gnu-tar` to install.
- If you're reverse proxying through a web server like `nginx`, you might be running different applications on the same server, but using `subpaths`. This is a very common scenario but took us some time to figure it out, so putting the steps here if you'd like to do the same:

Open `grafana.ini` and change the `[server].domain` config to your server name including the subpath.

Open `prometheus.service` in /etc/systemd/system and use `web.external.host` to set the right config.