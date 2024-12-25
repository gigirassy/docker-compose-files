On Github or somewhere else? This is mirrored from: https://gitz.blitzw.in/nune/docker-compose-files/
## Spec recommendations
50+ GB of space, 4 threads, and 12GB of RAM should be enough to run a large bulk of these services at once. However, some have done similar stuff to me with just 4GB of RAM.
## How to use
You need to install docker-compose. Also Caddy if you need a reverse proxy.
Generally with these, you'll make a folder for the service you wanna host, make a file called compose.yml, copy the contents in, modify accordingly, and `docker compose up -d` these up.
## Getting a domain
For a reverse proxy, you need a domain. If you can't buy any domain for any reason, eu.org and [freedns](https://freedns.afraid.org/) are awesome projects that can give you a subdomain for free that you can reverse-proxy safely to.
You can also use your real cash to pay via buying a prepaid card from behind the counter and using that to rent out domains.
### IMPORTANT
Make sure your domain has WHOIS privacy available 24/7 with no cost. Otherwise, you could easily be doxxed with a lookup of your domain.
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
Once installed, make a folder and make a file named Caddyfile. Write this in it with nano, and replace accordingly. The number can be replaced with the machine port (again, first number under ports section). Repeat this step for any new services.
Make sure the domain has a DNS record with the server IP.
```
example.com {
    reverse_proxy localhost:7825
}
```
Then type `caddy reload`.
Wait a bit, and your service should come up.
You might also want to slap this at the beginning of your Caddyfile and replace accordingly; it links your email so you get notified when your SSL certificates are about to expire.
```
{
    email me@example.com
}
```
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
cd etc/cron-daily
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

## Misc
### Fail2ban w/ AbuseIPDB
Fail2ban is a handy little program for Linux that will ban people trying to brute-force into your systems.

To help other sysadmins out, make an AbuseIPDB account and follow instructions to verify your account and get an API key in your settings.

Install fail2ban with: `apt-get install fail2ban`

Then, go to your root directory and type in the following commands.

```
cd etc/fail2ban
sudo nano jail.local
```

Paste the following text in and replace YOUR-API-KEY:

```
[sshd]
enabled = true
port    = ssh
maxretry = 3
findtime = 60d
bantime = 7d
ignoreip = 127.0.0.1
logpath = /var/log/auth.log

action = %(action_)s
         %(action_abuseipdb)s[abuseipdb_apikey="YOUR-API-KEY", abuseipdb_category="18,22"]
```

This should automatically ban and report those who try to get into your server. Don't be shocked at the bulk of reports you'll automatically upload to AbuseIPDB; those bots are literally everywhere, spamming random IPs 24/7.

### Uptime checking w/ Updown.io

While there is open source status maintaining software out there, they're not really suited to a novice's needs. This option costs money eventually, but it's ultimately very cheap and worth it, and they will send a notification to your email immediately if your service is down or has an expired SSL certificate.

Setting this up is very simple, make an account with your most active email and then add your sites' URLs accordingly. Set it to check every 5 or 10 minutes. They support basic auth, which is great.

Then, make a status page (Update it every time you add a new link!) and link it to a domain if you want.