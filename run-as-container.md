# Run the API as a docker container 
A `docker cli client` is installed inside this API. This docker client will contact the target docker server to get the information of that server. This is a clear-text TCP communication on port `2375`. Which means the docker daemon should be exposed on port `2375` at the server.

Following steps can be followed to expose the the docker daemon on `port 2375`.

Edit/create the file `/etc/docker/daemon.json`

```
{
"metrics-addr" : "0.0.0.0:9323",
"experimental" : true,
"hosts": [ "unix:///var/run/docker.sock", "tcp://0.0.0.0:2375" ]
}
```

This will expose the metrics of the docker server on `9323` and you can connect to docker host locally by socket file and remotely by listening port `2375`. 

Please note: Anybody who can reach the docker server, will be able to access the docker daemon without any authentication as TLS is not activated here.

Now this will create a configuration conflict as we are specifying a different daemon address from the default. Docker listens on a socket by default. On Debian and Ubuntu systems using `systemd`, this means that a host flag `-H` is always used when starting `dockerd`. If you specify a hosts entry in the `daemon.json`, this causes a configuration conflict and Docker fails to start.

To work around this problem, create a new file `/etc/systemd/system/docker.service.d/docker.conf` with the following contents, to remove the -H argument that is used when starting the daemon by default.

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```
Restart the docker and verify the listening port of docker on Ubuntu

`service docker restart`

`systemctl daemon-reload`

```
[ubuntu@localhost ~]# sudo netstat -ntlp | grep dockerd
tcp6       0      0 :::2375                 :::*                    LISTEN      1382/dockerd        
tcp6       0      0 :::9323                 :::*                    LISTEN      1382/dockerd

```

More detail can be found [here](https://docs.docker.com/config/daemon/#configure-the-docker-daemon "Docker Doc").
