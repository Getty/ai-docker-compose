# Copy to target

Copy all files to your target directory, I normally take */srv*

# Steps

 * Copy *sample.env* to *.env* and fill up the fields

 * Edit *traefik/traefik.toml* and add an email, you can switch to production lets encrypt, if tried with staging.

 * Change the users in traefik/config/20_auth.toml with *htpasswd -n username*

# Firewall

You need to assure that everything is masqueraded and you need to move the traffic of 80 and 443 to the traefik docker, here a sample *nftables.conf*, 1.1.1.1 needs to be replaced with the server IP and 10.31.0.250 with the traefik IP on the docker network.

```
#!/usr/sbin/nft -f

flush ruleset

table inet filter {

  chain input {
    type filter hook input priority 0
  }

  chain forward {
    type filter hook forward priority 0
  }

  chain output {
    type filter hook output priority 0
  }

}

table ip nat {

  chain postrouting {
    type nat hook postrouting priority 0
    ip saddr 10.0.0.0/8 masquerade
  }

  chain prerouting {
    type nat hook prerouting priority -100
    ip daddr 1.1.1.1 tcp dport { 80, 443 } dnat to 10.31.0.250
  }

}
```

# Docker Config

This needs to be copied to */etc/docker/daemon.json* or adapted with new IPs if 10.31.x.x is not used.

```json
{
    "bip": "10.31.1.1/24",
    "default-address-pools": [
        {
            "base": "10.31.128.0/17",
            "size": 24
        }
    ],
    "fixed-cidr": "10.31.1.1/24",
    "ip-forward": false,
    "ip-masq": false,
    "iptables": false,
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    },
    "userland-proxy": true
}
```

# Start Docker

 * *docker compose pull*

 * *docker compose up -d*

