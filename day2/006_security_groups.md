### Security groups

#### Table of contents

1.	Introduction
2.	Structure
3.  Security group scopes
4.  Rules evaluation sequence
4.  Creating security groups
5.  Binding security groups
6.  Viewing security groups
7.  Managing security groups

#### Introduction

Security groups are a very convenient way to allow certain communication channels to be open for some services.
They work very much like a firewall to control outbound traffic from your deployed applications.

### Creating security groups

To create security groups, the CLI provides the `create-security-group` command.

First, let's create a security group that will allow access to a fictional MySQL server, running on our local instance:

```sh
echo '[{"protocol":"tcp", "destination":"0.0.0.0/24","ports":"3306"}]' > mysql-sg.json
```

Now that you have created the file, you can use the CLI:

```sh
cf create-security-group mysql-sg mysql-sg.json
```

The output will show that the security group was created correctly:

```sh
$ cf create-security-group mysql-sg mysql-sg.json
Creating security group mysql-sg as admin
OK
```
### Binding security groups

We have successfully created a security group. How do we enable it, so that it works?

To do this, we need to *bind* the security group to either a space or a security group set.

#### Binding to spaces

Easy!

```sh
cf bind-security-group mysql-sg my-org my-first-space
```
> **Tip**: A space may belong to more than one application security group.

The output should be:

```
$ cf bind-security-group mysql-sg my-org my-first-space
Assigning security group mysql-sg to space my-first-space in org my-org as admin...
OK



TIP: Changes will not apply to existing running applications until they are restarted.
```

#### Binding to security group sets

To create a rule to be applied to every space in every org of your deployment, bind the group to a security group set.

Depending on what security group set you want to use, there are two different commands (do not run these commands yet, we will get back to them later on):

```sh
cf bind-staging-security-group mysql-sg
```

or

```sh
cf bind-running-security-group mysql-sg
```
### Viewing security groups

The CLI offers a wide variety of ways to display information about our security groups:

- To view all the security groups in the current org, use the `cf security-groups` command.
- To view the security groups bound to the **Default Staging** set, use the `cf staging-security-groups` command.
- To view the security groups bound to the **Default Running** set, use the `cf running-security-groups` command.
- To view detailed information about a security group, use the `cf security-group SECURITY-GROUP-NAME` command. In our example, the command will be `cf security-group mysql-sg`:

```sh
$ cf security-group mysql-sg
Getting info for security group mysql-sg as admin
OK

Name    mysql-sg
Rules
	[
		{
			"destination": "0.0.0.0/0",
			"ports": "3306",
			"protocol": "tcp"
		}
	]

     Organization   Space
#0   my-org         my-first-space
```
### Managing security groups

#### Updating

To update a security group, use the following command:
```sh
cf update-security-group SECURITY_GROUP_NAME FILE_PATH
cf update-security-group mysql-sg mysql-sg.json
```

This will update an existing security group with the new rules.

#### Deleting a security group

`cf delete-security-group SECURITY_GROUP_NAME` will delete a security group. Remember that to delete a security group, you must first unbind it.

#### Unbinding and deleting

The CLI offers several commands for unbinding and deleting security groups:

- `cf unbind-security-group SECURITY_GROUP_NAME ORG SPACE` will unbind a security group from a space
- `cf unbind-staging-security-group SECURITY_GROUP_NAME` will unbind a security group from the set of security groups for staging applications
- `cf unbind-running-security-group SECURITY_GROUP_NAME` will unbind a security group from the set of security groups for running applications

#### Try it

Let's unbind the security group we have created and then delete it:

1. `cf unbind-security-group mysql-sg my-org my-first-space`
2. `cf delete-security-group mysql-sg`
### Prerequisites

Before starting this lesson, plese create new org and space. Run cf `cf create-org my-org`, then run `cf target -o my-org` to get into your new org and `cf create-space my-first-space`. 

#### Structure

A security group consists of a predetermined structure, which is defined by a JSON object:

```js
[
  {
    "protocol":"tcp",
    "destination":"10.0.11.0/24",
    "ports":"3306"
  },
  {
    "protocol":"udp",
    "destination":"10.0.11.0/24",
    "ports":"2200-3000"
  }
]
```

The structure is defined by:

* **Protocol**: TCP, UPD, or ICMP
* **Destination**: an IP address or CIDR block
* **Ports**:
  * For TCP and UPD, a port to open or a port range
  * For ICMP, an ICMP type and code
#### Security groups scopes

Security groups can affect *spaces*, the **Default Staging**, or the **Default Running** group set:

- *Default Staging*: Rules in this set are applied to every application staged anywhere in your CF deployment.
- *Default Running*: Rules in this set are applied to every application running anywhere in your CF deployment.

> **Remember**: Security groups created and assigned in a Cloud Foundry deployment are **NOT** the same as infrastructure-level security groups.

#### Rules evaluation sequence

Multiple security groups can be applied to a space or a security set. Any of those security groups can allow an outgoing message. Since the security group rules define allowed traffic, the order of execution is not important, because Cloud Foundry merges the security groups applicable to the container and blocks all the traffic not allowed by the security group rules.
