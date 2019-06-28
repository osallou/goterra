# Goterra

## Status

In development

## About

Goterra is a Terraform based deployment tool, for multi cloud deployment of virtual machines.
It allows to create define some recipes (shell scripts) to execute in VM and to deploy an infra via Terraform templates.
Help with goterra store component, VMs can exchange data during deployment to be used by any VM plan or after deployment to get some result ouputs (a cluster token for example)

* Authentication

Users are created/authenticated against the *auth* service. Once binded via an API Key, user receives a *fernet* token that can be used against all goterra services. This token has a lifetime (24h by default).

* Create a namespace (shared by multiple users)

User creating the namespace is the namespace owner and can add other user ids as an owner or a member. See APIs for allowed access between owners and members.

In the namespace

* Create endpoints, endpoint represents a cloud where VMs can be deployed (could be multiple openstack clouds, amazon, ...)
* Create recipes, a recipe is a shell script executed in VM at startup. Recipe can extend an other recipe and is versioned.
* Create applications, an application is a set of recipes to apply, and a template for each kind of endpoint (a terraform template for openstack and an other for Amazon for example)

Endpoints, recipes and applications can define a set of variables that will be defined at runtime. Those variables are used in application templates (see Terraform variables) and also injected in recipes as environment variables.

Then user can run an application, specifying an endpoint and a set of variable. Variables will be written to a *variable.tf* file which can be used in Terraform templates (for user connection for example, choice of flavor, etc.). API also offers some *sensitive inputs* which are managed differently (not stored like other variables but injected via environement variables only). *sensitive inputs* are not accessible by recipes, only by terraform templates.

Recipes and applications can be set as *public* to make them available to other namespaces.

## License

Apache 2.0

## Components

* [goterra-store](https://github.com/osallou/goterra-store)
* [goterra-deploy](https://github.com/osallou/goterra-deploy)
* [goterra-run-agent](https://github.com/osallou/goterra-run-agent)
* [goterra-auth](https://github.com/osallou/goterra-auth)
* [goterra-ui](https://github.com/osallou/goterra-ui)

The store component can be deployed as stand-alone app (with auth service for authentication, or alone for anonymous access) to be used as an exchange service between VM/services/... See [goterra-store](https://github.com/osallou/goterra-store) for more info.

Each component has its own API, defined in openapi.yaml (Open API 3.0)

Web interface provides an API view of all components.

All components can be scaled up/down independently.

## Architecture

![alternative text](http://www.plantuml.com/plantuml/proxy?src=https://raw.github.com/osallou/goterra-docker/master/architecture.txt)

## Setup/Run

### From source

Compile each binaries or get the latest available release for each component then execute them along a goterra config file. Config file can be specified with env variable GOT_CONFIG

To communicate between them, components need an intermediate proxy. Auto discovery can be set with Traefik+Consul, but nginx/apache setup can be done.

* For simple config with only 1 instance for each component one can specify the GOT_PROXY_AUTH env variable to tell services where auth service is located.
* For configurations with a proxy, specify the proxy address with the env variable GOT_PROXY

The *deploy* and *run-agent* service communicates via RabbitMQ. Specify amqp connection url in goterra.yml file.

#### Requirements

* Mongodb
* RabbitMQ
* Consul (optional)
* Traefik (optional)

### Docker

Create a .env with variables:

* GOT_SECRET: secret to use for token generation
* GOT_DIR: directory where compose and configuration files are located

Use docker-compose to set up whole goterra

Traefik address/port is the external entrypoint. Services register to consul which is used by Traefik for service discovery.

## User registration

Users can auto-register via Google authentication if enabled.

User can self-register via API if allowed in configuration (via env variable **GOT_FEATURE_SELF_REGISTER**)

## Access

All traffic is HTTP, but of course, an HTTPS proxy should be put in front of Goterra.

Default route **/** should be redirected to **/app** to access Web UI.

## Roadmap

See GitHub (project)[https://github.com/osallou/goterra/projects] 