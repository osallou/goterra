# Goterra

## Status

In development

## About

goterra is a Terraform based deployment tool, for multi cloud deployment of virtual machines.

* Create a namespace (shared by multiple users)

In namespace

* Create endpoints, endpoint represents a cloud where VMs can be deployed (could be multiple openstack clouds, amazon, ...)
* Create recipes, a recipe is a shell script executed in VM at startup. Recipe can extend an other recipe and is versioned.
* Create applications, an application is a set of recipes to apply, and a template for each kind of endpoint (a terraform template for openstack and an other for Amazon for example)

Endpoints, recipes and applications can define a set of dynamic variables that will be defined at runtime. Those variables are used in application templates (see Terraform variables).

Then user can run an application, specifying an endpoint and a set of dynamic variable. Dynamic variables will be written to a variable.tf file which can be used in Terraform templates (for user/password connection for example, choice of flavor, etc.).

Recipes and applications can be set a *public* to make them available to other namespaces.

## License

Apache 2.0

## Components

* goterra-store
* goterra-deploy
* goterra-auth

## Run

Use docker-compose to set up whole goterra

Traefik address/port is the external entrypoint. Services register to consul which is used by Traefik for service discovery.

## Architecture

![alternative text](http://www.plantuml.com/plantuml/proxy?src=https://raw.github.com/osallou/goterra-docker/master/architecture.txt)



