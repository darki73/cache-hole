# Cache Hole

This project aims to provide you with the ability to run Pi-Hole and Lancache DNS servers on the same machine using Docker.

# General Information

In order to provide you with the adblocking capabilities of Pi-Hole and the game caching capabilities of Lancache, this project uses Docker to run both services on the same machine.

The idea behind this implementation is to allow you to run Pi-Hole as the main DNS server on your network, use Lancache for resolving game server addresses and on top of that, use DoH (DNS over HTTPS) to resolve all other DNS queries which is handled by Cloudflare's daemon.

1. Pi-Hole runs on the default DNS port 53 and listens on all interfaces.
2. The upstream server of the Pi-Hole is set to the Lancache DNS server.
3. The upstream server of the Lancache DNS server is set to the Cloudflare daemon.

This setup allows you to have a single DNS server on your network that can handle all DNS queries and resolve them accordingly.

# Installation

## Installing Docker
You need to have Docker installed on your machine in order to run this project. You can install Docker by following the instructions provided by the Digital Ocean: [Docker Installation on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04/)

## Cloning the project
You can clone the project by running the following command:
```bash
git clone https://github.com/darki73/cache-hole cache-hole
```
Project comes with folder that hosts volumes for Pi-Hole and DNSMasq configurations.

## Configuring the project

There are two configuration files (`.env`) which you can edit to suit your needs.  
Most of the values may be left as default, but some of them you have to change in order for the project to work properly.

### `.env` - global configuration

These are the variables that you might want to change:
1. `TZ` - Timezone of the container. You can find the list of available timezones [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). Remeber to use the value from the `TZ Indetifier` column.
2. `CUSTOM_DNS_NETWORK_ADDRESS` - The IP address of the new network which will be created to ensure each container can communicate with each other. You can leave it as default.
3. `CUSTOM_DNS_NETWORK_CIDR` - The CIDR of the new network which will be created to ensure each container can communicate with each other. You can leave it as default.

### `lancache.env` - Lancache DNS configuration

These are the variables that you have to change:
1. `LANCACHE_IP` - the IP address of your Lancache server.

#### Enabling / Disabling Lancache services
Resolving all of the available services through the Lancache might be an overkill, especially when some of them are not used, or the content provider switched from HTTP to HTTPS (in this case, the Lancache will not cache the content).  

As of 15 of April 2024, the following services are known to be working with Lancache (the ones i have tested):
1. Steam
2. Epic Games
3. Blizzard

Each of the services comes with two environmental variables that you can set in order to configure how Lancache DNS will behave.

For example, when we are talking about Steam, you have option to configure the following variables:
1. `DISABLE_STEAM` - If set to `true`, Lancache DNS will not route requests to Steam through the Lancache Server.
2. `STEAMCACHE_IP` - This variable holds the list of IP addresses of your Lancache servers. If left empty, it will use the value set in the `LANCACHE_IP` variable. If you have multiple servers, you may specify them by separating them with a space. For example: `STEAMCACHE_IP="192.168.100.100 192.168.100.101 192.168.100.102"`

## Configuring Pi-Hole
The main configuration (resolvers) is already done using the environmental variables in the `docker-compose.yaml` file, but you may want to change some of the settings in the Pi-Hole web interface.

### Changing the password
The default password for the Pi-Hole web interface will be automatically generated.  
You can set the password manually in the `docker-compose.yaml` file by adding the `WEBPASSWORD` to the list of existing `environment` variables in the `pihole` service.

Alternatively, you can change the password in the Pi-Hole web interface by following these steps:
```
docker exec -it pihole bash
pihole -a -p
```

As we have mapped the volumes to the host machine, the changes you make in the Pi-Hole web interface or CLI will be saved on the host machine.