# Day 3 excercises

## 1 - Fix the deployment

1. Get the manifest file:
`curl -o cf_cli.deb -J -L 'https://cli.run.pivotal.io/stable?release=debian64&source=github'`

2. Replace all the occurences of the values:
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

5. Once Cloud Foundry has been deployed, deploy the `cf-example-sinatra` applicatiob:
`cf push my-sinatra-app`

6. Now, try to tail the logs:
`cf logs my-sinatra-app`

Nice error, huh?

**Your mission**: try to fix the `manifest.yml` file by applying all the platform troubleshooting techniques learned in class.

## 2 - Create your own buildpack
