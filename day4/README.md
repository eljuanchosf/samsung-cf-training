# Day 4 excercises

## 1 - Concourse CI

1. SSH into your jumpbox
1. Get the Concourse CI CLI: 
        
        wget -O fly https://github.com/concourse/concourse/releases/download/v3.3.1/fly_linux_amd64

1. Make it available: 

        chmod +x fly && mv fly /usr/local/bin/

1. Login as an admin:

        fly --target samsung login --insecure --concourse-url https://concourse-up-samsung-9340854.eu-west-1.elb.amazonaws.com --username admin --password gbuxd0jn9g0k5ln4df0d

1. Create your own team

        fly set-team --target samsung -n team_YOUR_TEAM_NAME \
            --basic-auth-username ci \
            --basic-auth-password YOUR_PASSWORD

1. Login with your user:

        fly login --target samsung -n team_YOUR_TEAM_NAME --insecure

1. Follow [this](https://github.com/starkandwayne/concourse-tutorial) tutorial, from lessons 1 to 16 inclusive.
1. To access the web interface, go to [this address](https://concourse-up-samsung-9340854.eu-west-1.elb.amazonaws.com/).