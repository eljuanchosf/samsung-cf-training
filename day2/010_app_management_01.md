### Creating a free Pivotal Web Services account

In order to complete this lesson and all the lessons for Developers, you will need a Pivotal Web Services account.
Don't worry, is free and no credit card is required!

1. Go to the [Pivotal Web Services Trial home](https://try.run.pivotal.io/homepage)
2. Enter your data.
3. They will send a verification email. Open your email and verify your account.
4. They will ask for a SMS validation, so select your country and enter your cell phone number.
5. You will receive your verification code.
6. Enter the verification code and press the button.
7. Your account is verified!
8. Create an organization. It has to be a unique name. For example: `your-name-org-pws`
9. Once your organization is created, use the Cloud Foundry CLI to login: 
```
cf login -a https://api.run.pivotal.io -u YOUR_EMAIL -p YOUR_PASSWORD`
```

> **VERY IMPORTANT NOTICE**: when deploying your application to Pivotal Web Services, a unique name must be used. So, when you see `MY_APP`, please replace this value with the name of the application you chose. Example of a unique name: `jpg-sinatra-example-app`
>

### Your first application

First, if you don't have git, you should install it using the following commands

```exec
sudo apt-get update
sudo apt-get install git -y
```

Before we can deploy anything to Cloud Foundry, we need source code. Get it by downloading the example Sinatra app, like this:

```exec
cd ~/deployment
git clone https://github.com/Altoros/cf-example-sinatra
```

Next you should create new organization and space:

```exec
cf target -o YOUR_ORG
cf create-space my-space
```

And finally, you should target you organization and space using `cf target` command.

```exec
cf target -o YOUR_ORG -s my-space
```

> **Tip:** If you want to list all avaliable organizations use `cf orgs` commaand. All spaces can be listed after you already target your organization with `cf target -o MY_ORG_NAME` command. You can use `cf spaces` command in order to do this.
### Pushing applications

The act of deploying an application to Cloud Foundry is called **pushing**. Therefore, the command that does this is called `push`.

Once logged in, you can simply use the `cf push` command and one parameter (application name) to deploy the source code to Cloud Foundry.

```exec
cd cf-example-sinatra
cf push my-sinatra-app
```

This short command initiates a series of processes that upload your code to Cloud Foundry, detect the language used, download the corresponding [buildpack](http://docs.cloudfoundry.org/buildpacks/) (we will go deeper into buildpacks further on), and run the necessary scripts and commands to get the libraries used in the application.

To see your first application running, open a browser and navigate to the URL that Cloud Foundry has assigned to it.

#### Controlling resources when pushing

As you can see, the `cf push` command uses some default values, such as  disk size and memory limits, when pushing applications. To specify custom values, the CLI offers a number of modifiers:

```exec
cf push my-sinatra-app -k 128M -m 256M
```

This command will push your application again, setting a disk size (`-k`) of 128M and a memory limit (`-m`) of 256M. Run `cf help push` to get help on all the modifiers available for the `push` command.

```
$ cf help push
NAME:
   push - Push a new app or sync changes to an existing app

ALIAS:
   p

USAGE:
   Push a single app (with or without a manifest):
   cf push APP_NAME [-b BUILDPACK_NAME] [-c COMMAND] [-d DOMAIN] [-f MANIFEST_PATH]
   [-i NUM_INSTANCES] [-k DISK] [-m MEMORY] [-n HOST] [-p PATH] [-s STACK] [-t TIMEOUT]
   [--no-hostname] [--no-manifest] [--no-route] [--no-start]

   Push multiple apps with a manifest:
   cf push [-f MANIFEST_PATH]


OPTIONS:
   -b                   Custom buildpack by name (e.g. my-buildpack) or GIT URL (e.g. 'https://github.com/heroku/heroku-buildpack-play.git') or GIT BRANCH URL (e.g. 'https://github.com/heroku/heroku-buildpack-play.git#develop' for 'develop' branch)
   -c                   Startup command, set to null to reset to default start command
   -d                   Domain (e.g. example.com)
   -f                   Path to manifest
   -i                   Number of instances
   -k                   Disk limit (e.g. 256M, 1024M, 1G)
   -m                   Memory limit (e.g. 256M, 1024M, 1G)
   -n                   Hostname (e.g. my-subdomain)
   -p                   Path to app directory or file
   -s                   Stack to use (a stack is a pre-built file system, including an operating system, that can run apps)
   -t                   Maximum time (in seconds) for CLI to wait for application start, other server side timeouts may apply
   --no-hostname        Map the root domain to this app
   --no-manifest        Ignore manifest file
   --no-route           Do not map a route to this app and remove routes from previous pushes of this app.
   --no-start           Do not start an app after pushing
   --random-route       Create a random route for this app
```
### Creating application manifests

Although pushing source code to Cloud Foundry is really simple, you will probably need some place to specify and save all the parameters you customized when deploying your application. As you can see from the help that the `cf push` command offers, there is something called a *manifest*. A manifest is a [YAML](http://yaml.org/) file that usually resides in the root directory of your source code. If you name it `manifest.yml` or `manifest.yaml`, the CLI will pick it up automatically without you having to specify its location.

Now, create a manifest template, using the following command:

```exec
cf create-app-manifest my-sinatra-app
```

The CLI has just created a `my-sinatra-app_manifest.yml` file, which looks like this:

```yaml
---
applications:
- name: my-sinatra-app
  memory: 256M
  instances: 1
  host: my-sinatra-app
  domain: YOUR_CF_DOMAIN
```
### Pushing an application with a manifest

Now, let's reduce the amount of instance memory again, since our application is very small and can run on 128M without any issues. Open the manifest file and change the `memory:` value to `512M`. Save it and push the application again, this time, specifying the manifest file:

```exec
cf push -f my-sinatra-app_manifest.yml
```

As you can see, it is much more convenient to use an application manifest for pushing apps.

Here is one more trick to make pushing code even easier: rename the `my-sinatra-app_manifest.yml` file to `manifest.yml`. Then, if you do `cf push` with no parameters at all, the CLI will pick the `manifest.yml` file automatically.

Go ahead and try it!
### Application management

Cloud Foundry's CLI provides a myriad of options for managing your applications.

#### Listing applications

In the same way that orgs, users, and spaces can be listed, there is an option to list the deployed apps:

```sh
cf apps
```

You will see your application listed in the output.

> **Tip:** `cf apps` will only list applications that were pushed into the *target* (the org and space we have set before). If you want to list applications in another org/space, you will need to set the target again with the `cf target` command.

Getting detailed information about the application is as easy as:

```
cf app my-sinatra-app
```
#### Modifying applications

One of the possible scenarios you may run into is that you will need to rename your application. Again, this is *very* simple:

```exec
cf rename my-sinatra-app MY_APP
```

Now, if you run the `cf app MY_APP` command, you will see that the application has been renamed. This, however, will not update your `manifest.yml` or the route (URL) to access your app. What Cloud Foundry does here is modify the internal name, so you can use that name in your CLI commands.
#### Deleting applications

Another command you will find useful is `delete`. It effectively removes an application from Cloud Foundry.

```exec
cf delete MY_APP -f
```

> **Tip:** You can always skip the confirmation, using the `-f` flag: `cf delete MY_APP -f`.

