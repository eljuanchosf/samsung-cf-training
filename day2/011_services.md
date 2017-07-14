### Table of contents

1.  Introduction
2.  The Service Marketplace
3.  Creating services
4.  Binding and unbinding services
5.  Managing services
6.  User-provided services

### Introduction

No application is an island. At some point, every piece of code out there in the cloud uses external software. A database, a message queue, a mailing server... choose your flavor.
Cloud Foundry makes it extremely easy to use external software, referred to as *services* in the Cloud Foundry lingo.
### The Service Marketplace

Getting information about the available services is vital if you need to know which ones are available for consumption.
Cloud Foundry provides a catalog of services that you can explore and review, called the *Service Marketplace*.
To access it, just do:

```exec
cf marketplace
```

The output will show the name of the service, available plans, and a description of what the service does.
A *service plan* is a set of limits and rules that the service states. It can be a disk size limit for a database, a memory limit for a queue management system, or any set of limits that uniquely identifies a subset of a service.
In this example, the plans for the `cleardb` (a MySQL database) service are:
* spark
* boost
* amp
* shock

Although useful, this information will not show if the service is free or paid. There is another way to view the details of a service:

```exec
cf marketplace -s cleardb
```

### Creating services

To use a service plan and *instantiate* it, you need to create a *service*. A *service* is an instance of a *service plan* that you can use in your application.
Let's create a database for our application:

```exec
cf create-service cleardb spark MY_APP-db
```

The parameters of the `create-service` command are:

* the name of the software service
* the plan identifier
* the name of *your* service (This name is very important, since it is the one you are going to use for your application.)

You can explore which services are instantiated by doing:

```exec
cf services
```

Also, getting detailed information for a service is very easy:

```exec
cf service MY_APP-db
```

### Binding and unbinding services

So, how do you use the service you have just created in your application?
To do this, you need to *bind* your service to your application.
Binding will allow your app to get the necessary information to use the service. This includes the connection URI, username, password, connection parameters, etc.

```exec
cf bind-service MY_APP MY_APP-db
```

Now, let's try it in our application.

First, you need to have some code in place to use the service.
Go to the `cf-example-sinatra` application that you have cloned before, and do:

```exec
git checkout with-cleardb
```

Now, use your favorite text editor to open the `app.rb` file. If you are not comfortable using console editor you can use File Browser included on the page. In order to edit files using the File Browser, right click on the file and select 'Edit content' option.

You will notice this chunk of code:

```ruby
vcap_services = JSON.parse(ENV['VCAP_SERVICES'])
db_user = vcap_services['cleardb'].first['credentials']['username']
db_password = vcap_services['cleardb'].first['credentials']['password']
db_host = vcap_services['cleardb'].first['credentials']['hostname']
@db_name = vcap_services['cleardb'].first['credentials']['name']
```

This is where all the magic is hidden.
The first line will parse the JSON object that Cloud Foundry injects into the `VCAP_SERVICES` environment variable.
The following lines will access the properties of that JSON object to get the necessary information to connect to the database and retrieve some data.

Now, push your app with `cf push MY_APP`.

If you go to the URL provided by Cloud Foundry, you will see some information extracted from MySQL's *information schema*.

That's it!

Of course, there is the `unbind-service` command, which will detach a service instance from an application. Do **NOT** unbind the service right now, you will do it later.

```
cf unbind-service MY_APP my-sinatra-app-db
```
### Managing services

Services, of course, can mutate over time. Cloud Foundry's CLI provides a series of commands to manage those changes.
#### Renaming services

If you want to change the name of your service, run:

```exec
cf rename-service MY_APP-db my-sinatra-app-db
```

This will rename your service instance.
>**WARNING**: Beware that, if you are discovering a service instance by its name, you will have to update your code and push your app again.
#### Updating services

A service update will upgrade or downgrade the service instance plan.

```exec
cf update-service my-sinatra-app-db -p 1gb
```

Now, your service has been upgraded to the much larger `1gb` plan. You can verify it by doing `cf serice my-sinatra-app-db`.
>**WARNING**: Although the CLI has the `update-service` command by default, not all services support it. Please check the documentation of your services.
#### Deleting services

To delete a service, simply use the `delete-service` command:

```
cf delete-service my-sinatra-app-db
```

Hey, what happened? The command failed. Well, this is a safeguard. Cloud Foundry cannot delete a service that is still bound to an app. So, please **do** unbind the service as stated above and then, you can do `cf delete-service my-sinatra-app-db`. The  proper command sequence is:

```exec
cf unbind-service MY_APP my-sinatra-app-db
cf delete-service my-sinatra-app-db
```

There you go!

Now, as a little exercise, please create the `MY_APP-db` service instance again with the **512mb** plan and bind it to the `MY_APP` application.
### User-provided services

A *user-provided service* is a neat way in which Cloud Foundry allows you to use external software (applications that aren't being managed in Cloud Foundry) and treat them just like any other CF managed service.

Let's say you have a MySQL database that is external to the Cloud Foundry deployment. Your application needs to connect to that database, so you need to provide the connection information and credentials. Hard coding that information is not only **discouraged**, but is actually very dangerous. A user provided service will provide your application with a way to connect to that service without having to change the way you get the connection information.
#### Creating a user-provided service

Let's create a fictional service:

```
cf cups my-fictional-service -p "host, port, db_name, username, password"
```
> **TIP**: Since the `create-user-provided-service` command is a little long, we are using its alias, `cups`. It's shorter and easier to read!

Note that the CLI will ask you for values for each key that you put between the quotes.
If you want to create the service non-interactively, just do:

```exec
cf cups my-other-fictional-service -p '{"host":"the-db-host","db_name":"the-db-name","username":"the-user-name","password":"the-super-secret-user-password","db_port":"3306"}'
```

Now, bind the service to the `MY_APP` application, just like any other regular service with `cf bind-service MY_APP my-fictional-service`.
If you execute `cf env MY_APP`, you will be able to see the service credentials under the `user-provided` key, ready to use!
Also, check out what the user-provided services look like when you do `cf services` and `cf service my-fictional-service`.
#### Modifying a user-provided service

Now, let's say you need to modify the user-provided service with a new key to specify the port:

```exec
cf uups my-fictional-service -p '{"host":"the-db-host","db_name":"the-db-name","username":"the-user-name","password":"the-super-secret-user-password","db_port":"3306"}'
```
>**Tip**: `uups` is an alias for `update-user-provided-service`. There is no interactive mode for this command.

Now, unbind the service from the `MY_APP` application and bind it again. Check the `cf env MY_APP` result, and you will see the changes there!

Do you want to see it in action?
Go to the `cf-example-sinatra` directory and do:

```exec
git checkout with-ups
```

Now, do 
```exec
cf bs MY_APP my-fictional-service 
cf push MY_APP
```
>**Tip**: Instead of using `bind-service`, we are using its alias, `bs`.

Go to the URL that CF provides and you will see the credentials you have just entered with some service metadata.
#### Deleting a user-provided service

Deleting a user provided service is as easy as deleting a regular service:

```exec
cf delete-service my-other-fictional-service -f
```

And that's it!!
