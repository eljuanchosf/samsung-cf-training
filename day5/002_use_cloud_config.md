### Create Cloud Config

The [cloud config](https://bosh.io/docs/cloud-config.html) is a YAML file that defines IaaS specific configuration used by the Director and all deployments. It allows us to separate IaaS specific configuration into its own file and keep deployment manifests IaaS agnostic.

Now we are going to extract IaaS specific information from our greeter release and put it into the cloud config. We also will use the cloud config in all further deployments. Our initial cloud config will look ilke the following. Lets save this file as `~/deployment/cloud-config.yml`

```file=~/deployment/cloud-config.yml
azs:
- name: z1
  cloud_properties: {availability_zone: {{source ~/deployment/vars && echo $avz}} }

vm_types:
- name: t2.small
  cloud_properties:
    instance_type: t2.small
- name: t2.large
  cloud_properties:
    instance_type: t2.large
    ephemeral_disk: {size: 30000, type: gp2}
    root_disk: {size: 50000, type: gp2}

networks:
- name: private
  type: manual
  subnets:
  - range: 10.0.0.0/24
    gateway: 10.0.0.1
    az: z1
    static: [10.0.0.7 - 10.0.0.10]
    reserved: [10.0.0.2 - 10.0.0.6] 
    dns: [10.0.0.2]
    cloud_properties:  
      subnet: {{source ~/deployment/vars && echo $subnet_id}} 
      security_groups: [greeter, bosh]
- name: public
  type: vip

compilation:
  workers: 5
  reuse_compilation_vms: true
  az: z1
  vm_type: t2.large
  network: private

```

Next we need to upload cloud config to the Director.
```exec
bosh -n update-cloud-config ~/deployment/cloud-config.yml
```
### Update deployment manifest

When useing cloud config manifest schema is different. See [here](https://bosh.io/docs/manifest-v2.html) for details. Manifest for our greeter-release will look like the following. You need to save this file as `~/deployment/greeter.v2.0.yml`

```file=~/deployment/greeter.v2.0.yml
name: greeter-release
director_uuid: {{source .profile && bosh env | grep UUID | awk '{print $2}'}}

releases:
- name: greeter-release
  version: latest

instance_groups:
- name: app
  instances: 1
  azs: [z1]
  vm_type: t2.small
  stemcell: ubuntu
  jobs:
  - name: app
    properties: {}
  networks:
  - name: private
    static_ips: 
    - 10.0.0.7

- name: router
  instances: 1
  azs: [z1]
  vm_type: t2.small
  stemcell: ubuntu
  jobs:
  - name: router
    properties:
      upstreams:
        - 10.0.0.7:8080
  networks:
  - name: private
    static_ips:
    - 10.0.0.8
    default: [dns, gateway]
  - name: public
    static_ips:
    - {{source deployment/vars && echo $eip_greeter}}

stemcells:
- alias: ubuntu
  os: ubuntu-trusty
  version: {{source .profile && bosh stemcells | grep aws | awk '{print $2}' | sed -e 's/\*$//'}}

update:
  canaries: 1
  canary_watch_time: 30000
  update_watch_time: 30000
  max_in_flight: 10
  max_errors: 1
```

The following changes were made:

1.`compilation` and `networks` sections were moved to cloud config.
1. `stemcells` section was added
1. `jobs` section now becomes `instance_groups` and following changes also were applied 
  * `azs` property, that is responsible for configuring availability zoned were added.
  * former `templates` block now is renamed to `jobs`  
  * `properties` now are defined as a subsection of `jobs` They should be defined for each job separately
  * Instead of `resource_pool` we now specify `vm-type` and `stemcell`
### Redeploy greeter release

Before we will be able to redeploy greeter release using cloud config, we need to delete the old deployment.

```exec
bosh -n -d greeter-release delete-deployment
```

Now lets redeploy our greeter release 

```exec
bosh -n -d greeter-release deploy ~/deployment/greeter.v2.0.yml
```

You can check if everything has been deployed as intended:
```exec
curl "http://{{source ~/deployment/vars && echo $eip_greeter}}:8080"
```
