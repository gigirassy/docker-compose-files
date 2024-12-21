On Github or somewhere else? This is mirrored from: https://gitz.blitzw.in/nune/docker-compose-files/
## How to use
You need to install docker-compose. Also Caddy if you need a reverse proxy.
Generally with these, you'll make a folder for the service you wanna host, make a file called compose.yml, copy the contents in, modify accordingly, and `docker compose up -d` these up.
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