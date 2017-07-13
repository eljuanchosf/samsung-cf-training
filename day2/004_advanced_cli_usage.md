### Objectives

In this section, you will learn:

* How to use `CF_TRACE` to diagnose deployment issues
* How to install, use, and uninstall some handy CLI plugins
* How to use `cURL` to retrieve valuable data for scripting
* How OAuth works

### Introduction

From time to time, you might face some issues when working with Cloud Foundry. You can troubleshoot these issues in a number of ways, but Cloud Foundry's CLI provides some convenient out-of-the box functionality, which makes finding and fixing bugs a lot easier.
### Prerequisites to the lesson

Install CF-CLI. To download the .deb package, use cURL:
```sh
curl -o cf_cli.deb -J -L 'https://cli.run.pivotal.io/stable?release=debian64&version=6.21.1&source=github-rel'
```

Then, to install it, simply run:

```sh
sudo dpkg -i cf_cli.deb
```
Check, whether CF-CLI has been successfully installed by typing: 

```sh
cf --version
```

In case of successfull installation, you will see the following output:

```sh
$ cf --version
cf version 6.14.0+2654a47-2015-11-18
```
Install git by running the following command:

```sh
sudo apt-get update
sudo apt-get install git -y
```
Сonnect to Cloud Foundry's API, using the cf api command from the previous lessons.

```sh
cf api --skip-ssl-validation https://api.YOUR_CF_DOMAIN
```
Finally provide your credentials using `cf login` command and use `admin / admin ` login and password.

### Using `CF_TRACE`

When deploying applications or sending commands to Cloud Foundry, sometimes, you will need to take a look at what is going on with your requests.
The CLI provides a mode in which all the requests are shown as output. Basically, a **very** verbose mode.

To enable this mode, add the `CF_TRACE=true` environment value to the command line. Try it!

```sh
CF_TRACE=true cf orgs
```

The output should be something like:


```
$ CF_TRACE=true cf orgs

VERSION:
6.10.0-b78bf10

Getting orgs as my-user...

.
[LOTS of output!]
.

name
my-org
```

This "ultra verbose mode" is useful when diagnosing issues with any command you might send to Cloud Foundry.
As you can see, there is a lot of output. The CLI has a handy convention to solve this issue, as well:

```sh
export CF_TRACE=/path/to/outputfile.txt
```

This will output all the trace from all the CLI commands to the specified file.
### CLI plugins

One of the coolest features of the CLI is the ability to develop and install plugins to perform different tasks that are not provided out-of-the-box.

#### Installing a plugin

To try out this feature, we will use the [CLI-recorder](http://github.com/simonleung8/cli-plugin-recorder) plugin, which provides a **very** handy feature — a possibility to record all the commands that you issue into a macro.

Plugins are installed from either a local path, a URL, or a remote repo registered with the CLI.

```sh
cf install-plugin CLI-Recorder -r CF-Community
```

>**Tip**: If you want to try the plugin we have just installed, the list of commands can be found <a href="https://github.com/simonleung8/cli-plugin-recorder#full-command-list" target="_blank">here</a>.

#### Managing plugins

* You can list all the installed plugins with `cf plugins`.
* To remove the plugin, do `cf uninstall-plugin CLI-Recorder`. Don't do it yet though, as we will soon need it!
* To list all the plugin repositories, do `cf list-plugin-repos`.
* To list all the available plugins for an installation, do `cf repo-plugins`.

#### Command collision

Plugin names and commands must be unique. If you install a plugin that has a command that collides with another command, the CLI will display an error message.

Solving this issue involves uninstalling the existing plugin and installing a new one.
### Using `cf curl`

Sooner or later, you will probably want to script some actions based on Cloud Foundry's responses.
In that case, the `cf curl` command is very useful. It can retrieve Cloud Foundry's responses and parse them to get data.
`cf curl` uses the good old `curl` command, setting the authorization data and the requests headers required by Cloud Foundry.

Try executing `cf curl /v2/apps`.

You will get a JSON object that contains all the information for all the deployed applications.

So, how de we use this data?
You can install the `jq` JSON parser by doing `sudo apt-get install jq`.
Now, you can parse the response with `jq`:

```sh
cf curl /v2/apps | jq .resources[].entity.name
```

The following script will get the application name indirectly:

```sh
#!/bin/bash
set -e

APP_URL=$(cf curl /v2/apps | jq -r ".resources[].metadata.url")
APP_NAME=$(cf curl $APP_URL | jq -r ".entity.name")

echo My App name is: $APP_NAME
```
### Getting a CLI authorization token

If you want to use a tool other than `cf curl` to access the Cloud Foundry API, you are going to need an authorization token.
Cloud Foundry issues one authorization token per every application that gets authenticated.
To demostrate how you can use this token, we will make a regular `curl` request.

Let us try the `curl` command without a token:

```sh
curl http://api.YOUR_CF_DOMAIN
```

It will return something like:

```
$ curl http://api.YOUR_CF_DOMAIN
{
  "code": 10002,
  "description": "Authentication error",
  "error_code": "CF-NotAuthenticated"
}
```

To authorize the request, you need to get a token:

```sh
cf oauth-token
```

This will return a token for the CLI's session:

```
$ cf oauth-token
Getting OAuth token...
OK

bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIwOWQwNzhiYS1lZmU5LTQ3YzgtYjJiNC0zNDE1NzljZGRlYmIiLCJzdWIiOiJkMWViNzg1OS1iZWFjLTQyMTctYWQzMy1lMzc3MzBiM2ZmYTYiLCJzY29wZSI6WyJjbG91ZF9jb250cm9sbGVyLnJlYWQiLCJwYXNzd29yZC53cml0ZSIsImNsb3VkX2NvbnRyb2xsZXIud3JpdGUiLCJvcGVuaWQiXSwiY2xpZW50X2lkIjoiY2YiLCJjaWQiOiJjZiIsImF6cCI6ImNmIiwiZ3JhbnRfdHlwZSI6InBhc3N3b3JkIiwidXNlcl9pZCI6ImQxZWI3ODU5LWJlYWMtNDIxNy1hZDMzLWUzNzczMGIzZmZhNiIsIm9yaWdpbiI6InVhYSIsInVzZXJfbmFtZSI6Imp1YW4ucGFibG8uZ2Vub3Zlc2VAYWx0b3Jvcy5jb20iLCJlbWFpbCI6Imp1YW4ucGFibG8uZ2Vub3Zlc2VAYWx0b3Jvcy5jb20iLCJyZXZfc2lnIjoiNTBiZDBmMCIsImlhdCI6MTQ1NjI2MjkxMCwiZXhwIjoxNDU2MzA2MTEwLCJpc3MiOiJodHRwczovL3VhYS5uZy5ibHVlbWl4Lm5ldC9vYXV0aC90b2tlbiIsInppZCI6InVhYSIsImF1ZCI6WyJjbG91ZF9jb250cm9sbGVyIiwicGFzc3dvcmQiLCJjZiIsIm9wZW5pZCJdfQ.KL6lQ2LPFhzyT8kymxuLa9qJ_XG1IDkg_-Dex9L3ce4
```

This authentication token is what we need to do our regular `curl` command:

```sh
curl -H 'Authorization: bearer eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIwOWQwNzhiYS1lZmU5LTQ3YzgtYjJiNC0zNDE1NzljZGRlYmIiLCJzdWIiOiJkMWViNzg1OS1iZWFjLTQyMTctYWQzMy1lMzc3MzBiM2ZmYTYiLCJzY29wZSI6WyJjbG91ZF9jb250cm9sbGVyLnJlYWQiLCJwYXNzd29yZC53cml0ZSIsImNsb3VkX2NvbnRyb2xsZXIud3JpdGUiLCJvcGVuaWQiXSwiY2xpZW50X2lkIjoiY2YiLCJjaWQiOiJjZiIsImF6cCI6ImNmIiwiZ3JhbnRfdHlwZSI6InBhc3N3b3JkIiwidXNlcl9pZCI6ImQxZWI3ODU5LWJlYWMtNDIxNy1hZDMzLWUzNzczMGIzZmZhNiIsIm9yaWdpbiI6InVhYSIsInVzZXJfbmFtZSI6Imp1YW4ucGFibG8uZ2Vub3Zlc2VAYWx0b3Jvcy5jb20iLCJlbWFpbCI6Imp1YW4ucGFibG8uZ2Vub3Zlc2VAYWx0b3Jvcy5jb20iLCJyZXZfc2lnIjoiNTBiZDBmMCIsImlhdCI6MTQ1NjI2MjkxMCwiZXhwIjoxNDU2MzA2MTEwLCJpc3MiOiJodHRwczovL3VhYS5uZy5ibHVlbWl4Lm5ldC9vYXV0aC90b2tlbiIsInppZCI6InVhYSIsImF1ZCI6WyJjbG91ZF9jb250cm9sbGVyIiwicGFzc3dvcmQiLCJjZiIsIm9wZW5pZCJdfQ.KL6lQ2LPFhzyT8kymxuLa9qJ_XG1IDkg_-Dex9L3ce4
' http://api.YOUR_CF_DOMAIN
```

Don't forget to replace the token with your own. Ok, yes, it is a bulky request, but it works!
