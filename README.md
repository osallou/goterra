# Goterra

## Status

In development

## About

Goterra is a Terraform based deployment tool, for multi cloud deployment of virtual machines.
It allows to create define some recipes (shell scripts) to execute in VM and to deploy an infra via Terraform templates.
Help with goterra store component, VMs can exchange data during deployment to be used by any VM plan or after deployment to get some result ouputs (a cluster token for example)
At run time, user specifies the value of the expected variables (specified in recipes or templates)

### Authentication

Users are created/authenticated against the *auth* service. Once binded via an API Key, user receives a *fernet* token that can be used against all goterra services. This token has a lifetime (24h by default).

### Create a namespace (shared by multiple users)

User creating the namespace is the namespace owner and can add other user ids as an owner or a member. See APIs for allowed access between owners and members.

In the namespace

* Create endpoints, endpoint represents a cloud where VMs can be deployed (could be multiple openstack clouds, amazon, ...)
* Create recipes, a recipe is a shell script executed in VM at startup. Recipe can extend an other recipe and is versioned.
* Create applications, an application is a set of recipes to apply, and a template for each kind of endpoint (a terraform template for openstack and an other for Amazon for example)

User information needed at runtime

* Endpoint secrets are created by end user to fill his credentials for related endpoint.
* User SSH key can be set in user profile or sent at run time in Run input *ssh_pub_key* variable.

Endpoints, recipes and templates can define a set of variables that will be defined at runtime. Those variables are used in application templates (see Terraform variables) and also injected in recipes as environment variables.

Then user can run an application, specifying an endpoint and a set of variable. Variables will be written to a *variable.tf* file which can be used in Terraform templates (for user connection for example, choice of flavor, etc.). API also offers some *sensitive inputs* which are managed differently (not stored like other variables but injected via environement variables only). *sensitive inputs* are not accessible by recipes, only by terraform templates.

Recipes, Endpoints, Templates and applications can be set as *public* to make them available to other namespaces.

### Execution

When an application is ran, user provides expected inputs, then run is sent to a queue. Once queue executes the run, VMs are deployed. Deployment is in 2 steps:

* Terraform plan: the VMs (with storage, etc.) are deployed, once done, run is in deploy_succes (or failure) status
* Once a VM is started, recipes are applied, their status can be obtained from goterra store component, or, in UI, seen in Recipes status. Each VM will have a specific recipe status (over or failed).

All run inputs are available as variables for Terraform plan AND recipes as environment variables.

A deployment is considered successfull only when both plan and recipes are in success. In case of recipe failure, some error logs are available. However VMs will not be stopped. User **should** ask for resources destruction in case of failure.

User can get all outputs (if any) defined by terraform plan directly from goterra *deploy* component.
Other data set in *store* by VMs should be queried directly in *store* component.

When run is stopped, all resources, including *store* data are deleted.

Only run *owner* can access store data.

### Store

*store* component can be used in terraform to exchange data between VMs.
Help with *goterra-cli* command, automatically installed in VMs, one can push or fetch data on the *store*.
In case of fetch, if requested key is not present, it will wait until a defined timeout to get it. After the timeout, it fails.

An example scenario is a master VM that generates a token, push it to the *store*. During this time a slave VM fetches this token, waiting for it to be available. Once available it takes its value for its own setup and connect to the master with the token.

## License

Apache 2.0

## Components

* [goterra-store](https://github.com/osallou/goterra-store): store to handle VMs communication and store deployment status/progress
* [goterra-deploy](https://github.com/osallou/goterra-deploy): component to handle namespaces,recipes,etc...
* [goterra-run-agent](https://github.com/osallou/goterra-run-agent): terraform execution agent
* [goterra-auth](https://github.com/osallou/goterra-auth): authentication and user management
* [goterra-ui](https://github.com/osallou/goterra-ui): web interface
* [goterra-acct](https://github.com/osallou/goterra-acct): accounting
* [goterra-cli](https://github.com/osallou/goterra-cli): command line client

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
* GOT_URL: url to the service

Use docker-compose to set up whole goterra

Traefik address/port is the external entrypoint. Services register to consul which is used by Traefik for service discovery.

## User registration

Users can auto-register via Google authentication if enabled.

User can self-register via API if allowed in configuration (via env variable **GOT_FEATURE_SELF_REGISTER**)

## Access

All traffic is HTTP, but of course, an HTTPS proxy should be put in front of Goterra.

Default route **/** should be redirected to **/app** to access Web UI.

## Roadmap

See GitHub [project](https://github.com/osallou/goterra/projects)
