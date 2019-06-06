# goterra docker setup

Use docker-compose to set up whole goterra

Traefik address/port is the external entrypoint. Services register to consul which is used by Traefik for service discovery.

## Architecture

![alternative text](http://www.plantuml.com/plantuml/proxy?src=https://raw.github.com/osallou/goterra-docker/master/architecture.txt)



