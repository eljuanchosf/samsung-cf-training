### Objectives

In this section, you will learn:

* What the UAAC client is and why it is necessary in Cloud Foundry
* How to download and install Cloud Foundry's UAAC client
* What user management operations need the UAAC client
* How to issue basic and intermediate UAAC commands

### Introduction

The CF CLI is a wonderful tool to execute and resolve most of the every day tasks in a Cloud Foundry deployment.
However, there are some user authentication and authorization jobs that need the UUA Client to be performed.
### What is the UAAC?

The UAA (User Authentication \& Authorization) is a multi tenant identity management service used in Cloud Foundry. Its primary role is to serve as an OAuth2 provider, issuing tokens for client applications to use when they act on behalf of Cloud Foundry users. It can also authenticate users with their Cloud Foundry credentials and act as an SSO service, using those (or other) credentials. The UAA has endpoints for managing user accounts and for registering OAuth2 clients, as well as various other management functions.

The UAA**C** is the UAA CLI, which provides a wrapper for API exposed by the UAA.

#### Why is it necessary?

Cloud Foundry's CLI only provides very basic UAA actions, such as creating and deleting users and assigning permissions to orgs and spaces.
The UAAC allows for much more complex operations, such as general and specific system-wide user roles and permissions.

#### More info about the UAAC

If you'd like to go deeper into the UAAC, after completing all the lessons, you can explore the [UAAC GitHub repository](https://github.com/cloudfoundry/cf-uaac), which contains a lot of useful resources and information.
### Installing the UAAC

The UAAC is written in [Ruby](http://www.ruby-lang.org) and distributed as a [Ruby gem](https://en.wikipedia.org/wiki/RubyGems). Ruby has been pre-installed on your training jumpbox, so you don't have to install it manually.

To install the UAAC, you can do:

```sh
gem install cf-uaac
```

This will output something like:

```
$ gem install cf-uaac
Fetching: cf-uaac-3.1.7.gem (100%)
Successfully installed cf-uaac-3.1.7
Parsing documentation for cf-uaac-3.1.7
Installing ri documentation for cf-uaac-3.1.7
Done installing documentation for cf-uaac after 0 seconds
1 gem installed
```

Just to try it, type:

```sh
uaac help
```

It should show a lot of output with the commands and options the UAAC provides.

### Basic UAAC operations

#### Connecting to Cloud Foundry's UAA server

First, target your deployment:

```sh
uaac target uaa.YOUR_CF_DOMAIN --skip-ssl-validation
```
> **Tip**: We are targeting the `UAA` endpoint instead of the `API`. This is because the UAAC connects to and operates directly through the UAA component.
> **Warning**: Notice that, again, we are skipping SSL validation when connecting to the UAA. This is NOT recommended for real-life deployments.

For the next step, you will need to know the `uaa:admin:client:secret` property from Cloud Foundry's deployment manifest.
In this case, and for the sake of simplicity, we will simply provide it to you: `admin-secret`

Well, that "secret" is not much of a secret really, but we will learn how to change it in a different section.

The second step is to get authorized to use the UAAC:

```sh
uaac token client get admin -s admin-secret
```

This should output something like:

```sh
$ uaac token client get admin -s admin-secret

Successfully fetched token via client credentials grant.
Target: https://uaa.YOUR_CF_DOMAIN
Context: admin, from client admin
```
#### Checking permissions

To verify that our recently obtained security token has enough permissions to perform write operations to the UAAC, we need to do:

```sh
uaac contexts
```

And verify in the output that the `scope` property includes the `scim.write` permission.

#### Creating a secondary admin user

Let's create another user with admin permissions:

```sh
uaac user add MyAdminUser -p MySecretPassword --emails myemail@mydomain.com
```

The client will notify you if the user was successfully created.
Now, to grant the user admin permissions, run:

```sh
uaac member add cloud_controller.admin MyAdminUser
uaac member add uaa.admin MyAdminUser
uaac member add scim.read MyAdminUser
uaac member add scim.write MyAdminUser
```

To each one of these commands, the UAAC will respond with a `success` message.
Now, your new user is an `admin`!

#### Changing passwords

First, we need to confirm the admin user has enough permissions to change another user's password.

```sh
uaac context | grep scope
```

If you can't find the `password` value in the UAAC response, then you need to request it.
Replace the `MY-PERMISSIONS` text with the existing permissions from the output of the previous command.

```sh
uaac client update admin --authorities "MY-PERMISSIONS password.write"
```

Now, your `admin` user will be able to change passwords. However, you will need to delete the current token to get another one with the new permissions:

```sh
uaac token delete
uaac token client get admin -s admin-secret
```

Now, try:

```sh
uaac password set MyAdminUser -p AlwaysATempPassword
```

Next time `MyAdminUser` tries to sign in into the CLI with that password, they will be prompted for a new password.

Take a look at the [UAA scopes](https://docs.cloudfoundry.org/concepts/architecture/uaa.html#uaa-scopes) and what they mean.
