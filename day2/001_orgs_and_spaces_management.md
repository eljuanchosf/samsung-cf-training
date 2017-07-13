### Table of contents

1. Introduction
2. Managing organizations
3. Managing spaces

### Introduction

Out-of-the-box Cloud Foundry provides a powerful Role Based Access Control (RBAC) system that will help you to organize, secure, and provide flexible configurations for deploying applications and services.

To understand a little bit more about Orgs, Spaces, and Roles, you can read the [official documentation](https://docs.cloudfoundry.org/concepts/roles.html), which has a very good introduction.

In general, the structure can be defined as follows:

```
Organization(s)
|
--- Space(s)
    |
    --- Application(s)
```

In short, Cloud Foundry has one or many organizations, which in turn have one or many spaces. The users get permissions according to pre-defined roles, and the users deploy applications in the spaces assigned.

This simple, yet flexible organizational structure will allow you to adapt Cloud Foundry to your company's needs.

For now, we will focus on Organizations and Spaces, and later on, we will work with users and permissions.
### Managing organizations

To manage organizations, spaces, and assign permissions, you have to be logged in as an `admin` user, or a user with admin permissions.

To do this, you can use the `cf auth` command as we used it in the previous lesson (Quick Start):

```sh
cf auth admin admin
```

Once you are logged in, check your top-level organizational structure with:

```
cf orgs
```

This command will show you a list of organizations that you might have created previously.
#### Create organizations

Now, to add an organization, simpy do:

```sh
cf create-org my-org
```

You will see something like:

```
$ cf create-org my-org
Creating org my-org as admin...
OK

TIP: Use 'cf target -o my-org' to target new org
```

Now, try listing the existing organizations with `cf orgs` again.
Your new organization should be listed in the output.
#### Getting info about an organization

A very good way to get information about the organization you have just created is by using the `cf org` command:

```
cf org my-org
```

The result will be something like:

```
$ cf org my-org
Getting info for org my-org as admin...
OK

my-org:
          domains:        YOUR_CF_DOMAIN
          quota:          default (10240M memory limit, Unlimited instance memory limit, 1000 routes, 100 services, paid services allowed)
          spaces:
          space quotas:

```

>**Tip:** note the *singular* vs. the *plural* noun used in `cf orgs`. Using the plural will get you a list of objects represented by the noun, and the singular version will do something else, usually get details. This is a very common pattern in Cloud Foundry's CLI.
>
#### Modifying and deleting organizations

At some point, you might need to rename one of the Orgs you have previously created. The good news is that Cloud Foundry maintains an internal reference for the Org, decoupling the name from the internal structure:

```sh
cf rename-org my-org my-renamed-org
```

The output should be:

```
$ cf rename-org my-org my-renamed-org
Renaming org my-org to my-renamed-org as admin...
OK
```

Also, at some point, is probable that we are going to need to delete an organization. You can do it with:

```sh
cf delete-org my-renamed-org
```

The CLI is going to ask for a confirmation and then effectively erase the org:

```
$ cf delete-org my-renamed-org

Really delete the org my-renamed-org and everything associated with it?> yes
Deleting org my-renamed-org as admin...
OK
```
>**Warning:** Deleting an Org means that Cloud Foundry will delete ALL objects and applications depending of that Org. It is a very destructive procedure, so be very carefull with it.
>
#### Targeting organizations

Since we have deleted the Org in the previous step, we are going to need to create a new one:

```sh
cf create-org my-org
```

First, let's try listing all the spaces in that Org. The initial step is to *target* the Org you want to get the spaces from.
This will set a default Org for the CLI to work with.

```sh
cf target -o my-org
```

The output should be:

```
$ cf target -o my-org

API endpoint:   https://api.YOUR_CF_DOMAIN (API version: 2.43.0)
User:           admin
Org:            my-org
Space:          No space targeted, use 'cf target -s SPACE'
```

Now that the CLI knows which Org to work with by default, we can do:

```sh
cf spaces
```

This will return a list of the existing spaces in the Org:

```
$ cf spaces
Getting spaces in org my-org as admin...

name
No spaces found
```
#### Creating spaces

Spaces are a very convenient way of separating concerns and environments inside an Org. Managing Spaces is not different from managing Orgs.

Creating a space is as simple as:

```sh
cf create-space my-first-space
```

The CLI will offer detailed information about the space created:

```
$ cf create-space my-first-space
Creating space my-first-space in org my-org as admin...
OK
Assigning role SpaceManager to user admin in org my-org / space my-first-space as admin...
OK
Assigning role SpaceDeveloper to user admin in org my-org / space my-first-space as admin...
OK

TIP: Use 'cf target -o my-org -s my-first-space' to target new space
```

> **Tip:** Notice that, for many of its commands, the CLI offers help on what to do next, e.g., how to target the space you have just created.
>

Now, try listing all the spaces again with the `cf spaces` command.
#### Getting info about spaces

Getting info about a space is as simple as:

```sh
cf space my-first-space
```
This will return a good deal of info about the space we have specified.
Don't worry about what all that information means. We'll get to it later on.

```
$ cf space my-first-space
Getting info for space my-first-space in org my-org as admin...
OK

my-first-space
                 Org:               my-org
                 Apps:
                 Domains:           YOUR_CF_DOMAIN.xip.io
                 Services:
                 Security Groups:   public_networks, dns, services, load_balancer, user_bosh_deployments
                 Space Quota:

```
#### Modifying and deleting spaces

Let's create a new space:

```sh
cf create-space my-second-space
```

```
$ cf create-space my-second-space
Creating space my-second-space in org my-org as admin...
OK
Assigning role SpaceManager to user admin in org my-org / space my-second-space as admin...
OK
Assigning role SpaceDeveloper to user admin in org my-org / space my-second-space as admin...
OK

TIP: Use 'cf target -o my-org -s my-second-space' to target new space
```

Now, if we need to rename the space, its very easy to do:

```sh
cf rename-space my-second-space my-trash-space
```

```
$ cf rename-space my-second-space my-trash-space
Renaming space my-second-space to my-trash-space in org my-org as admin...
OK
```

Again, you don't need to worry about changing the name, since the internal reference is maintained without alteration.

Deleting a space is really easy, as well:

```sh
cf delete-space my-trash-space
```

```
$ cf delete-space my-trash-space

Really delete the space my-trash-space?> yes
Deleting space my-trash-space in org my-org as admin...
OK
```

> **Tip:** you can use the `-f` modifier in some commands that ask for confirmation to force the "yes" without having to type it interactively. It is dangerous, but very useful when scripting commands. In this case, it should be `cf delete-space my-trash-space -f`
>
