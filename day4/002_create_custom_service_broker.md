## Install PostgreSQL DB

1. Add PostgreSQL Apt Repository.
  ```exec
  sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
  wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
  ```

1. Install PostgreSQL.
  ```exec
  sudo apt-get update
  sudo apt-get install -y postgresql postgresql-contrib
  ```
1. Set root password
  ```
  sudo -u postgres psql
  ALTER USER postgres with encrypted password 'some_password';
  \q
  ```

1. Configure postgres to allow remote connections.
  ```exec
  sudo sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '\*'/g" /etc/postgresql/9.6/main/postgresql.conf 
  sudo bash -c 'echo "host  all  all 0.0.0.0/0 md5" > /etc/postgresql/9.6/main/pg_hba.conf'
  sudo service postgresql restart 
  ```

1. Save DB connection URL.
  ```exec
  mkdir ~/deployment
  pg_url=postgresql://postgres:some_password@YOUR_IP:5432/postgres

  echo "export pg_url=$pg_url" > ~/deployment/vars
  ```
## Install required packages

1. Install **golang**
  ```exec
  wget https://storage.googleapis.com/golang/go1.7.linux-amd64.tar.gz
  sudo tar -xvf go1.7.linux-amd64.tar.gz
  sudo mv go /usr/local
  sudo ln -sf /usr/local/go/bin/go /usr/bin/go
  ```

1. Configure your `$GOPATH` and save it to your `.profile`
  ```exec
  export GOPATH=~/go
  mkdir $GOPATH
  echo "export GOPATH=$GOPATH" >> ~/.profile
  echo "export PATH=\$PATH:\$GOPATH/bin" >> ~/.profile
  source ~/.profile
  ```

1. Install govendor tool
  ```exec
  go get -u github.com/kardianos/govendor
  ```
## Create base application structure

1. Create application directory
  ```exec
  mkdir -p ~/go/src/github.com/$USER/cf-postgresql-broker
  cd ~/go/src/github.com/$USER/cf-postgresql-broker
  ```

1. Create the `Procfile` file in the cf-postgresql-broker folder with the following content:
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/Procfile
  web: cf-postgresql-broker
  ```

1. Create the `manifest.yml` file with the following content:
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/manifest.yml
  ---
  env:
    GOPACKAGENAME: cf-postgresql-broker
  ```

## Install application dependencies

1. Download and install go packages.
  ```exec
  go get -u github.com/altoros/pg-puppeteer-go
  go get -u github.com/pivotal-cf/brokerapi
  go get -v github.com/cloudfoundry/lager
  mkdir $GOPATH/src/code.cloudfoundry.org/
  cp -R $GOPATH/src/github.com/cloudfoundry/lager/ $GOPATH/src/code.cloudfoundry.org/
  ```

1. Create the `main.go` file which is going to be the source file for our application:
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  package main

  import (
    "context"
    "encoding/json"
    "errors"
    "net/http"
    "os"

    "github.com/altoros/pg-puppeteer-go"
    "github.com/pivotal-cf/brokerapi"
    "code.cloudfoundry.org/lager"
  )
  ```
## Implement the main function

The **main** function is a starting point for any go application. Have a look at code comments.

``` sh
file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
```
This is a struct, that is responsible for handling all requests to the broker
```sh
type Handler struct {
  //DB operations orchestrator
  Db *pgp.PGPuppeteer
}
```
```sh
func main() {
	// Set up authentication
	credentials := brokerapi.BrokerCredentials{
		Username: os.Getenv("AUTH_USER"),
		Password: os.Getenv("AUTH_PASSWORD"),
	}

	//create database connection
	conn, err := pgp.New(os.Getenv("PG_SOURCE"))
	if err != nil {
		panic(err)
	}

	// Set up logger
	logger := lager.NewLogger("cf-postgresql-broker")
	logger.RegisterSink(lager.NewWriterSink(os.Stdout, lager.DEBUG))

	// Register requests handler
	http.Handle("/", brokerapi.New(&Handler{Db: conn}, logger, credentials))

	// Boot up
	port := os.Getenv("PORT")
	if err := http.ListenAndServe(":"+port, nil); err != nil {
		panic(err)
	}
}
```
## Implement service broker API methods

1. Implement the the **services** that returns the list of provided services
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  func (h Handler) Services(context context.Context) []brokerapi.Service {
    servicesJSON := `[{
      "id": "service-id",
      "name": "postgresql",
      "description": "DBaaS",
      "bindable": true,
      "plan_updateable": false,
      "plans": [{
        "id": "plan-id",
        "name": "basic",
        "description": "Shared plan"
      }]
    }]`
    services := make([]brokerapi.Service, 0)

    // Parse services list
    if err := json.Unmarshal([]byte(servicesJSON), &services); err != nil {
      panic(err)
    }
    return services
  }
  ```

1. Implement the **provision** method that creates a DB using instance ID.
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  func (h Handler) Provision(context context.Context, instanceID string, _ brokerapi.ProvisionDetails, _ bool) (brokerapi.ProvisionedServiceSpec, error) {
    dbname, err := h.Db.CreateDB(instanceID)

    if err != nil {
      return brokerapi.ProvisionedServiceSpec{}, err
    }

    return brokerapi.ProvisionedServiceSpec{
      IsAsync:      false,
      DashboardURL: dbname,
    }, nil
  }
  ```

1. Implement the **deprovision** method. It simply drops a DB.
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  func (h Handler) Deprovision(context context.Context, instanceID string, _ brokerapi.DeprovisionDetails, _ bool) (brokerapi.DeprovisionServiceSpec, error) {
    if err := h.Db.DropDB(instanceID); err != nil {
      return brokerapi.DeprovisionServiceSpec{}, err
    }

    return brokerapi.DeprovisionServiceSpec{
      IsAsync:      false,
    }, nil
  }
  ```

1. Implement the **bind** method. It is needed to create DB users for bound application.
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  func (h Handler) Bind(context context.Context, instanceID, bindingID string, _ brokerapi.BindDetails) (brokerapi.Binding, error) {
    creds, err := h.Db.CreateUser(instanceID, bindingID)

    if err != nil {
      return brokerapi.Binding{}, err
    }

    return brokerapi.Binding{
      Credentials: creds,
    }, nil
  }
  ```

1. Implement the **unbind** method. It drops users and all their privileges for a service.
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  func (h Handler) Unbind(context context.Context, instanceID, bindingID string, _ brokerapi.UnbindDetails) error {
    if err := h.Db.DropUser(instanceID, bindingID); err != nil {
      return err
    }

    return nil
  }
  ```

1. Implement the rest methods that we won't support but still they are required.
  ```file=~/go/src/github.com/$USER/cf-postgresql-broker/main.go
  func (h Handler) LastOperation(context context.Context, instanceID string, operationData string) (brokerapi.LastOperation, error) {
    return brokerapi.LastOperation{}, errors.New("Not supported") 
  }

  func (h Handler) Update(context context.Context, instanceID string, _ brokerapi.UpdateDetails, _ bool) (brokerapi.UpdateServiceSpec, error) {
    return brokerapi.UpdateServiceSpec{}, errors.New("Not supported")
  }
  ```

For more information about service broker API go to [the official documentation page](http://docs.cloudfoundry.org/services/api.html)
## Save dependencies

The `govendor` tool saves all your application dependencies into the `vendor` directory in order to implement the native vendoring

```exec
cd ~/go/src/github.com/$USER/cf-postgresql-broker
govendor init
govendor add +external
```
## Create the postgresql security group

1. Create the `~/deployment/postgresql.json` file with the following content:
  ```file=~/deployment/postgresql.json
  [
    {
      "destination": "0.0.0.0/0",
      "ports": "5432",
      "protocol": "tcp"
    }
  ]
  ```

1. Create the **postgresql** security group.
  ```exec
  cf create-security-group postgresql ~/deployment/postgresql.json
  ```

1. Bind the security group to your ORG and SPACE
  ```exec
  cf bind-security-group postgresql YOUR_ORG YOUR_SPACE
  ```
## Push the broker to the Cloud Foundry

1. Push code to the Cloud Foundry but without starting the application
  ```exec
  cd ~/go/src/github.com/$USER/cf-postgresql-broker
  cf push postgresql --no-start -m 128M -k 256M -b 'https://github.com/cloudfoundry/go-buildpack#v1.7.16'
  ```

1. Set application's environment. `{GUID}` will be replaced with runtime values.
  ```exec
  source ~/deployment/vars

  cf set-env postgresql PG_SOURCE "$pg_url"
  cf set-env postgresql AUTH_USER "admin"
  cf set-env postgresql AUTH_PASSWORD "admin"
  ```

1. Start the application
  ```exec
  cf start postgresql
  ```

1. Register it as a service broker
  ```exec
  BROKER_URL=$(cf app postgresql | grep urls: | awk '{print $2}')
  cf create-service-broker postgresql admin admin http://$BROKER_URL
  cf enable-service-access postgresql
  ```

1. Check installation in marketplace. `postgresql` should be there
  ```exec
  cf marketplace
  ```
## Push a test application

1. Download and install a test application.
  ```exec
  go get -u github.com/altoros/pg-app
  go get -u github.com/kardianos/govendor
  cd $GOPATH/src/github.com/altoros/pg-app
  govendor sync
  ```

1. Push it to the Cloud Foundry.
  ```exec
  cf push pg-app --no-start -m 128M -k 256M -b 'https://github.com/cloudfoundry/go-buildpack#v1.7.16'
  ```

1. Create a service
  ```exec
  cf create-service postgresql basic pgsql
  ```

1. Bind the application to the service.
  ```exec
  cf bind-service pg-app pgsql
  ```

1. Set the broker name environment variable
  ```exec
  cf set-env pg-app PG_BROKER_NAME postgresql
  ```

1. Start the application
  ```exec
  cf start pg-app
  ```

1. Verify the installation. You should see the `SELECT version()` query result:
  ```exec
  curl $(cf app pg-app | grep urls: | awk '{print $2}')
  ```
