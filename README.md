On Github or somewhere else? This is mirrored from: https://gitz.blitzw.in/nune/docker-compose-files/
## How to use
You need to install docker-compose. Also Caddy if you need a reverse proxy.
Generally with these, you'll make a folder for the service you wanna host, make a file called compose.yml, copy the contents in, modify accordingly, and `docker compose up -d` these up.
For a reverse proxy, you need a domain. If you can't buy any domain for any reason, eu.org and [freedns](https://freedns.afraid.org/) are awesome projects that can host stuff for you and give you a domain for free that you can reverse-proxy safely to.
## Things to keep in mind
The first four-five-digit number in ports is the port for your machine, the second after the `:` is the port for the container. Don't change the second number unless there's a configuration setting that allows you to modify like so.
Use a server in your home [properly configured](https://caddy.community/t/using-caddy-as-a-reverse-proxy-in-a-home-network/9427), or a VPS. I personally recommend Webdock for a VPS.
You don't need to do any steps to install dependencies for these services other than docker-compose, docker containers do that for you inside the container.
## Basic reverse proxy
This will work fine for most things assuming the URL is configured in `environment:` section/config if needed.
Install caddy with the following command after `apt-get update` and `apt-get upgrade`:
```
apt-get install caddy
```
Once installed, make a folder and make a file named Caddyfile. Write this in it with nano, and replace accordingly. The number can be replaced with the machine port (again, first number under ports section).
Make sure the domain has a DNS record with the server IP.
```
example.com {
    reverse_proxy localhost:7825
}
```
Then type `caddy reload`.
Wait a bit, and your service should come up.
## Keep your services up-to-date and save space!
To automatically keep images up-to-date, there's this handy thing called Watchtower that will update services automatically. To do this, it will stop the service, pull the new image, and put that image up. Your data will not be lost. This is extremely helpful. The command is below. (Run, not compose!)
```
docker run --detach \
    --name watchtower \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower
```

After that, the program will run in the background, updating images accordingly.

### However, there's one catch...

The old images don't get automatically deleted, which means they accumlate unnecessary space on your hard drive. You should set up a magical thing called a cron job.

Make sure you're root for this. In the root directory, do this...

```
cd /etc/cron-daily
nano docker-prune
```

Then, paste this code in and save. This will purge your old images every 3 days.

```
#!/bin/bash
docker system prune -af  --filter "until=$((3*24))h"
```

Now, do this command to make the file executable.

```
chmod +x /etc/cron.daily/docker-prune
```

And bam, the system will automatically detox itself, just like your liver.