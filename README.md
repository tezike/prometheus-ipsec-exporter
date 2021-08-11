# ipsec_exporter
Ipsec exporter to prometheus. This program creates an endpoint in that exposes
the state of ipsec tunnels.

There's [this go project](https://github.com/dennisstritzke/ipsec_exporter)
which does more or less the same as this project. It's main problem is that it
uses a way to test that the tunnel is up that doesn't work with Amazon Linux.
Amazon Linux uses `openswan` and that program is probably meant to be used with
`libreswan`.

Anyway, this project works correctly with both of them, since the way to check
that the tunnel is up is to ping the other side, not to execute a `ipsec`
command.

## Requirements

Docker installed:
```text
sudo yum install docker
```

Enable and Start the Docker Service
```
sudo systemctl enable docker
sudo systemctl restart docker
```

Python packages:

```text
flask>=1.0.2
prometheus-client>=0.5.0
```

Python versions:
- python2.7
- python3.5
- python3.6

But the tests only work with python3.

## Usage

```bash
docker run -p 9000:9000 --rm --name ipsec_exporter -d \
    -v /etc/ipsec.d/:/etc/ipsec.d/ -v \
    /var/run/pluto/pluto.ctl:/var/run/pluto/pluto.ctl \
    tezike/prometheus-ipsec-exporter
```

This will create an endpoint in the direction http://localhost:9000/metrics.

## Build docker image

There's two dockerfiles. One for Amazon Linux 1 image and the other for Amazon
Linux 2 image:

``` bash
docker login
docker build -f Dockerfile_2 -t paradigmadigitalorg/prometheus-ipsec-exporter:amazon-2 .
docker push paradigmadigitalorg/prometheus-ipsec-exporter:amazon-2
```

## Automatic deployment

To create a sysvinit script that uses the docker image, there's a playbook. You
must pass the docker registry url, username and password to download the image
from the instance. The credentials will be removed, so you don't have to worry
about that.

``` bash
ansible-playbook -i "$ip," -e ansible_ssh_user=$aws_user -e \
    docker_registry=$docker_registry -e \
    docker_username=$docker_username -e \
    docker_password=$docker_password ansible/main.yml
```

Note the comma after $ip, to use an inventory from the CLI you must use it like
this.

## Testing

``` bash
docker run -ti -v `pwd`:/tox/files/ alexperezpujol/tox:latest tox
```

## Third party

The script's source used to check if the tunnel is up may be checked in the
[zabbix-ipsec](https://github.com/a-schild/zabbix-ipsec) github repository. It's
a pretty cool option if you use zabbix.
