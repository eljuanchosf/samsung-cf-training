# Day 3 excercises

## 1 - Blue/Green deployment script

1. Copy the source code of the `cf-example-sinatra` application to the directory `cf-example-sinatra-green`:
`cp cf-example-sinatra cf-example-sinatra-green -R`
2. Open the `public/style.css` file and change the `header.background` property to `#00cc00`.

**Your mission**: create a `bash` script that will do blue/green deployment on the `cf-sinatra-example` application, deploying firs the "blue" version and then the "green" version.

## 2 - Fix the deployment

1. Get the manifest file:
`curl -o manifest.yml -J -L 'https://raw.githubusercontent.com/eljuanchosf/samsung-cf-training/master/day3/manifest.yml'`

2. Replace all the occurences of the following placeholders with the values for your deployment:
```
YOUR_CF_ELASTIC_IP
YOUR_AVAILABILITY_ZONE
YOUR_BOSH_DIRECTOR_UUID
YOUR_SUBNET_ID
```

3. Set the manifest
`bosh deployment manifest.yml`

4. Deploy!
`bosh -n deploy`

5. Once Cloud Foundry has been deployed, deploy the `cf-example-sinatra` application:
`cf push my-sinatra-app`

6. Now, try to tail the logs:
`cf logs my-sinatra-app`

Nice error, huh?

**Your mission**: try to fix the `manifest.yml` file by applying all the platform troubleshooting techniques learned in class.

## 3 - Create your own Buildpack!

Follow the instructions in [this tutorial](001_create_buildpack.md).

**Your mission**: have a custom buildpack ready!
