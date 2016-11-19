concourse-ssh-deploy
========================================

[aecepoglu/concourse-ssh-deploy on Docker Hub](http://hub.docker.com/r/aecepoglu/concourse-ssh-deploy)


A Concourse resource that you can use to deploy docker images in ssh'ed hosts

Source Configuration
-----------------------

* `private_key`: ssh key required for login

Behaviour
---------

### Check

doesn't do anything

### In

doesn't do anything

### Out

#### Params:

* `user`: *REQUIRED* ssh username
* `host`: *REQUIRED* ssh host
* `from_image`: *REQUIRED* docker image to deploy
* `port_mapping`: *OPTIONAL* port mapping for docker image. Accepts the same format as docker run. For example: `["80", "3001:4003"]
* `build_url_for_port`: *OPTIONAL* if given a port number, then the url in the form of `"http://{host}:{build_url_for_port}"` is written inside the file *info/url*. The example below should help explain.

Sample
---------

The pipeline below gets code from git repository, builds a docker image from it and pushes it to a registry, and then deploys that image so it's running at 192.168.2.100:3003

    resource_types:
    - name: ssh-deploy-target
      type: docker-image
      source:
        repository: aecepoglu/concourse-ssh-deploy
    
    resources:
    - name: ssh-target
      type: ssh-deploy-target
      source:
        private_key: {{private_key}}
    
    jobs:
    - name: build-and-deploy
      plan:
      - put: ssh-target
        params:
          user: user123
          host: 192.168.2.100
          port_mapping: ["80", "3001:3000"]
          from_image: dockercloud/hello-world
          build_url_for_port: 80
      - task: "view build info"
        config:
          platform: linux
          image_resource:
            type: docker-image
            source: {repository: busybox, tag: "1.25"}
          inputs:
          - name: ssh-target
          run:
            path: sh
            args:
            - -exc
            - |
              ls ssh-target
              ls ssh-target/info
              cat ssh-target/info/url

The output of the task **view build info** is:

    + ls ssh-target
    info
    + ls ssh-target/info
    url
    + cat ssh-target/info/url
    http://192.168.2.100:32781
