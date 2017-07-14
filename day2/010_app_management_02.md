### Table of contents

1.	Introduction
2.	Restarting applications
3.	Restaging applications
4.	Scaling up and down
5.	Restarting an instance
6.	Environment variables

### Introduction

Since we have deleted the application, let's deploy it again. But first, edit the `manifest.yml` file and change **only** the `name` attribute value to `MY_APP`. Then save the file and push the application:

```exec
cf push
```

Our application is running again! Yes, it's *that* easy.

### Restarting applications

There are several ways to restart an application, each one with some differences that make them very useful.

Try:

```exec
cf stop MY_APP
```

Check that the application has been halted successfully with `cf app MY_APP`. This will simply stop the application, leaving all the source code there, ready to be started again:

```exec
cf start MY_APP
```

The output will show that the application has been restarted.

These two steps can also be combined, like so:

```sh
cf restart MY_APP
```
### Restaging applications

There is another way to "restart" an application, which is called *restaging*. Restaging an application means that, except for uploading the application code, all the processes that are executed when pushing the application will run again. Let's have a look at what happens when you run `cf restage`:

```exec
cf restage MY_APP
```

As you can see, there is a fundamental difference between restaging and restarting applications: a simple `restart` will keep the droplet, whilst `restage` will erase the current droplet and create a new one with the previously pushed code.
### Scaling horizontally

Try scaling your application up to two instances:

```exec
cf scale MY_APP -i 2
```

Now, check the status of the application:

```sh
cf app MY_APP
```

You can see that there are now two instances of `MY_APP` running!
### Scaling vertically

First, let's scale the memory limit of the `MY_APP` application:

```exec
cf scale MY_APP -m 512M -f
```

Since this is a *very* small application that barely consumes any memory, we can safely shrink memory usage down to 64M:

```exec
cf scale MY_APP -m 64M -f
```

After the application has been restarted, check the status with the `cp app MY_APP` command. You should see that both instances have been restarted successfully.
### Scaling up and down

Scaling applications deployed to CF up and down is easy:

```
$ cf help scale
NAME:
   scale - Change or view the instance count, disk space limit, and memory limit for an app

USAGE:
   cf scale APP_NAME [-i INSTANCES] [-k DISK] [-m MEMORY] [-f]

OPTIONS:
   -i   Number of instances
   -k   Disk limit (e.g. 256M, 1024M, 1G)
   -m   Memory limit (e.g. 256M, 1024M, 1G)
   -f   Force restart of app without prompt
```

Cloud Foundry gives users full control over the size and number of their application instances. However, you must take the following into consideration:

* Scaling up and down can be done with the same command.
* Scaling vertically (modifying any of the instance size parameters) will *restart* the application.
* Scaling horizontally will NOT restart the application.
* The number of instances that can be created is limited by the *quotas* and *space quotas* you set.

**One more very important thing to know about how Cloud Foundry works with scaling applications:** when an application that has many instances is restarted, they get restarted in a sequence. Why? Simply to get zero-downtime deployments. When an instance of your application is down, the Cloud Foundry Router will NOT route packages to that instance. Simple and effective!
### Restarting an instance

Sometimes an instance may get interrupted, which may actually be normal behaviour justified by a number of reasons.
To restart it and have it working again, you can use the `cf restart-app-instance` command.

Let's say instance *#2* of your application has stopped for some reason. Restart it with:

```exec
cf restart-app-instance MY_APP 2
```

### Environment variables

Following the **12 factor app** principles, Cloud Foundry implements a number of enviroment variables that you, as a developer, can read and use. Please read the following section from the Cloud Foundry docs before moving forward: [Cloud Foundry Environment Variables](https://docs.cloudfoundry.org/devguide/deploy-apps/environment-variable.html).
#### Viewing variables

To view the environmental variables available to your application, use:

```exec
cf env MY_APP
```

#### Setting a variable

Let's assume that you need to set a variable to provide a value to your application. Just run:

```exec
cf set-env MY_APP MYVAR myvalue
```

Although this variable will be immediately availabe to all your application instances, if you want your application to start reading the variable, you need to **restage** it. However, this is not a compusory step and you can skip it.

Running `cf env MY_APP` again will reveal that `MYVAR` and its value appear under the `User-Provided` section.
Now, define the second variable:

```exec
cf set-env MY_APP MY_OTHER_VAR myothervalue
```

Check the variables again. To unset (remove) a variable from the environment, do:

```exec
cf unset-env MY_APP MY_OTHER_VAR
```

Easy as it seems, it is very powerful.
Also, you can set all of these variables in the application manifest, as we will see later on.
