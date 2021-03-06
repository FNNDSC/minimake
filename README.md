# ![ChRIS logo](https://raw.githubusercontent.com/FNNDSC/ChRIS_ultron_backEnd/master/docs/assets/logo_chris.png) miniChRIS

[![CI](https://github.com/FNNDSC/miniChRIS/workflows/CI/badge.svg)](https://github.com/FNNDSC/miniChRIS/actions?query=workflow%3ACI)
[![GitHub license](https://img.shields.io/github/license/FNNDSC/miniChRIS)](LICENSE)

Run a demo of ChRIS. https://chrisproject.org/

```bash
git clone https://github.com/FNNDSC/miniChRIS.git
cd miniChRIS
./minichris.sh
```

## Usage

A default superuser `chris:chris1234` is created.

website        | URL
---------------|-----
ChRIS_ui       | http://localhost:8020/
ChRIS admin    | http://localhost:8000/chris-admin/
ChRIS_store_ui | http://localhost:8021/

### Start

```bash
docker-compose up -d
```

### Stop

```bash
docker-compose down -v
```

### Fancy Start

Waits for the backend to be ready and does a few sanity checks.

```bash
./minichris.sh
```

### Not Working?

1. Make sure you have `docker` and `docker-compose` both installed and working properly.
2. Stop all running containers.
3. No process should be bound to ports 5005, 5010, 5055, 8000, 8010, 8020, 8021

#### Troubleshooting

Run `docker-compose down -v && docker-compose up -d`


If you still get an error, create a detailed [bug report](https://github.com/FNNDSC/miniChRIS/issues).
Include the output from the above command. Also wait 5 minutes and use
`docker-compose ps` to see specifically which service was unsuccessful, and
`docker-compose logs <service_name>` for a complete error report.

### Add Plugins

See [plugins/README](plugins/)

# Github Actions

*miniChRIS* can be used as a step in Github Actions workflows to spin up
an ephermeral instance of the ChRIS backend and its ancillary services
for the purpose of end-to-end testing.

```yaml
on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: Do nothing useful
    steps:
    - name: setup CUBE
      id: cube
      uses: fnndsc/miniChRIS@v5
    - name: make a request
      run: curl -u "${{ steps.cube.outputs.cube-user }}" "${{ steps.cube.outputs.cube-url }}"
```

### Examples

- [FNNDSC/cookicutter-chrisapp/.github/workflows/test.yml](https://github.com/FNNDSC/cookiecutter-chrisapp/blob/16db74860e8201f3d201183961eadc39116ce8a7/.github/workflows/test.yml#L31) uses *ChRIS miniChRIS* for end-to-end testing.
- [FNNDSC/cni-store-proxy/package.json](https://github.com/FNNDSC/cni-store-proxy/blob/master/package.json) uses *ChRIS miniChRIS* as a git submodule for a local dev environment.


# About

`./minichris.sh` is a no-nonsense collection of scripts to start ChRIS without the shenanigans of
[make.sh](https://github.com/FNNDSC/ChRIS_ultron_backEnd/blob/master/make.sh).
It is fully managed by `docker-compose`.

Traditionally, to bring up CUBE+pfcon+pman on a single-machine on-the-metal requires a few extra steps on the host.

CUBE setup involves:

1. waiting for web server to come online
2. creating a superuser
3. adding `host` as a compute environment
4. registering some plugins

### pman

`pman` setup involves:

1. joining a docker swarm
2. figuring out the [`STOREBASE` environment variable](https://github.com/FNNDSC/ChRIS_ultron_backEnd/blob/78670f6abf0b6ebac7aeef75989893b4502d4823/docker-compose_dev.yml#L208-L222)

`pman` is special because it itself is a container which spawns other containers on its host.

It needs `/var/run/docker.sock` to be mounted inside the container.
We can resolve the two setup requirements by connecting to the host's dockerd.

#### `STOREBASE`

The workaround for `STOREBASE` is not merged upstream because it might not work
on a multi-node docker swarm deployment using `docker stack deploy`.
The pull request is here: https://github.com/FNNDSC/pman/pull/165

Hence, a fork of `pman` with the patch applied is independently maintained and pushed
under the docker tag `fnndsc/pman:*-rc.?-single`.
This image is recommended for use with single-machine deployments.

### Performance

`./minichris.sh` takes 50 seconds on an okay laptop (quad-core, 16 GB, SSD)
and takes 2-3 minutes in [Github Actions' Ubuntu VMs](https://github.com/FNNDSC/miniChRIS/actions).
It is strongly recommended that you use an SSD!

### Goals

- fast
- simple use
  - no arguments
  - do one thing, and one thing well (a UNIX philosophy)
- legible code
- practical for E2E testing

#### Non-Goals

- configurable
- production use
- back-end development environment

### Vagrant

No docker? That's okay.

`Vagrantfile` provides a virtual machine (VM) with latest `docker-compose` installed.
VMs are inherently slow, and docker image pulls are not cached between lifecycles.
Setup using Vagrant is wasteful of time, memory, disk, and network bandwidth.
For me, `vagrant up` took 9 minutes.

#### Start

```bash
vagrant up
```

#### Stop

```bash
vagrant destroy -f
```
