# edX Devstack

Get up and running quickly with edX services.

If you are seeking info on the Vagrant-based devstack, please see https://openedx.atlassian.net/wiki/display/OpenOPS/Running+Devstack.
This project is meant to replace the traditional Vagrant-based devstack with a multi-container approach driven by [Docker
Compose](https://docs.docker.com/compose/). It is still in the alpha/beta testing phase. Support for this project is
limited at the moment, so it may take a while to respond to issues.

You should run any Make targets described below on your local machine, _not_ from within a VM.

## Prerequisites

This project was developed and tested using **Docker 1.13+**. If you are using OS X, please use [Docker for Mac](https://docs.docker.com/docker-for-mac/).
Previous Mac-based tools (e.g. boot2docker) are not supported.

[Docker for Windows](https://docs.docker.com/docker-for-windows/) may work but has not been tested and is _not supported_.


## Getting Started

All of the services can be run by following the steps below. Note that since we are running many containers, you should
configure Docker with a sufficient amount of resources. Our testing found that [configuring Docker for Mac](https://docs.docker.com/docker-for-mac/#/advanced)
 with 2 CPUs and 4GB of memory works well.

1. The Docker Compose file mounts a host volume for each service's executing code. The host directory is expected to be
  a sibling of this directory. For example, if this repo is cloned to `~/workspace/devstack`, host volumes will be
  expected in `~/workspace/course-discovery`, `~/workspace/ecommerce`, etc. These repos can be cloned with the command
  below.

  ```
  $ make clone
  ```

2. Run the provision command, if you haven't already, to configure the various services with superusers (for
  development without the auth service) and tenants (for multi-tenancy).

  The username and password for the superusers are both "edx". You can access the services directly via Django admin
  at the `/admin/` path, or login via single sign-on at `/login/`.

  ```
  $ make devstack.provision
  ```

3. Start the services.

  ```
  $ make devstack.start
  ```

After the services have started, if you need shell access to one of the services, run `make devstack.open.<service>`.
For example to access the Catalog/Course Discovery Service, you can run:

```
$ make devstack.open.discovery
```

### Docker Sync

Docker for Mac has known filesystem issues that significantly decrease performance. In order to mitigate these issues,
we use [Docker Sync](https://github.com/EugenMayer/docker-sync/wiki) to synchronize file data from the host machine to
the containers. Follow the steps below to setup Docker Sync.

1. Ensure all containers are stopped.
```
$ make devstack.stop
```

2. Follow the installation instructions at https://github.com/EugenMayer/docker-sync/wiki to install Docker Sync.

3. Run Docker Sync. This will be a separate process, so feel free to run this in a separate terminal window. Note that
you may see errors about permissions on `node_modules` sub-directories. These can be safely ignored.
```
$ docker-sync start
```

4. Start devstack.
```
$ make devstack.start
```


## Usernames and Passwords

The provisioning script creates a Django superuser for every service.

    Email: edx@example.com
    Username: edx
    Password: edx

The LMS also includes demo accounts. The passwords for each of these accounts is `edx`.


| Username | Email                |
|----------|----------------------|
| audit    | audit@example.com    |
| honor    | honor@example.com    |
| staff    | staff@example.com    |
| verified | verified@example.com |


## Service URLs

Each service is accessible at `localhost` on a specific port. The table below provides links to the homepage of each
service. Since some services are not meant to be user-facing, the "homepage" may be the API root.

| Service           | URL                                                                    |
|-------------------|------------------------------------------------------------------------|
| Credentials       | [http://localhost:18150/api/v2/](http://localhost:18150/api/v2/)       |
| Catalog/Discovery | [http://localhost:18381/api-docs/](http://localhost:18381/api-docs/)   |
| E-Commerce/Otto   | [http://localhost:18130/dashboard/](http://localhost:18130/dashboard/) |
| LMS               | [http://localhost:18000/](http://localhost:18000/)                     |
| Studio/CMS        | [http://localhost:18010/](http://localhost:18010/)                     |


## Useful Commands

Sometimes you may need to restart a particular application server. Rather than restarting the entire devstack or the
individual container, you can instruct [Supervisor](http://supervisord.org/) to restart just the nginx and application
servers with one command.

If you are already working in the shell of a container run:

```
$ /edx/app/supervisor/venvs/supervisor/bin/supervisorctl restart all
```

Alternatively, if you are not working from a shell inside a container run:

```
$ docker exec -t edx.devstack.<service>  bash -c '/edx/app/supervisor/venvs/supervisor/bin/supervisorctl restart all'
```

`<service>` should be replaced with one of the following:

* credentials
* discovery
* ecommerce
* edxapp

## Remaining Work

There is still work to be done before this is ready for full release to the Open edX community. Here are the major items:

1. Align with [OEP-5](http://open-edx-proposals.readthedocs.io/en/latest/oep-0005.html)
2. Finish provisioning all services
3. Load demo data
4. PyCharm debugging
5. Merge devstack settings for edxapp (https://github.com/edx/edx-platform/pull/14376), and reactivate host volume
