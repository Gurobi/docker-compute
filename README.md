![Logo](https://www.gurobi.com/wp-content/uploads/2018/12/logo-final.png "Gurobi Optimization")
# Quick reference
Maintained by: [Gurobi Optimization](https://www.gurobi.com)

Where to get help: [Gurobi Support](https://www.gurobi.com/support/), [Gurobi Documentation](https://www.gurobi.com/documentation/)

# Supported tags and respective Dockerfile links

* [9.5.0, latest](https://github.com/Gurobi/docker-compute/blob/master/9.5.0/Dockerfile)
* [9.1.2](https://github.com/Gurobi/docker-compute/blob/master/9.1.2/Dockerfile)

When building a production application, we recommend using an explicit version number instead of the `latest` tag.
This way, you are in control of the upgrade process of your application.

# Quick reference (cont.)

Supported architectures: linux amd64

Published image artifact details: https://github.com/Gurobi/docker-compute

Gurobi images:
- [gurobi/optimizer](https://hub.docker.com/r/gurobi/optimizer): Gurobi Optimizer (full distribution)
- [gurobi/python](https://hub.docker.com/r/gurobi/python): Gurobi Optimizer (Python API only)
- [gurobi/python-example](https://hub.docker.com/r/gurobi/python-example): Gurobi Optimizer example in Python with a WLS license
- [gurobi/modeling-examples](https://hub.docker.com/r/gurobi/modeling-examples): Optimization modeling examples (distributed as Jupyter Notebooks)
- [gurobi/compute](https://hub.docker.com/r/gurobi/compute): Gurobi Compute Server
- [gurobi/manager](https://hub.docker.com/r/gurobi/manager): Gurobi Cluster Manager

# What is `gurobi/compute`?
The Gurobi Optimizer is the fastest and most powerful mathematical programming solver available 
for your LP, QP and MIP (MILP, MIQP, and MIQCP) problems.
More info at the [Gurobi Website](https://www.gurobi.com/products/gurobi-optimizer/).

The Gurobi Compute Server has been designed to simplify the task of building and deploying modern 
optimization applications. It allows you to seamlessly offload your optimization 
computations onto one or more dedicated optimization servers grouped in a cluster. 
Users and applications can share the servers thanks to advanced queuing and load 
balancing capabilities. Users can monitor jobs, and administrators can manage the servers.
[Gurobi Compute Server Documentation](https://www.gurobi.com/documentation/current/remoteservices/index.html)

The `gurobi/compute` image provides a Docker image ready to be deployed in a cluster.

## Getting a Gurobi license

The [Web License Service](https://www.gurobi.com/web-license-service/) (WLS) is a new Gurobi licensing service 
for containerized environments (Docker, Kubernetes...). Gurobi components can automatically request and renew license tokens to 
the WLS servers available in several regions worldwide. WLS only requires that your container has access to the 
Internet. Commercial users can request an evaluation and academic users can request a free license.
Please register to access the [Web License Manager](https://license.gurobi.com) and read the
[documentation](https://license.gurobi.com/manager/doc/overview)

Note that other standard license types (NODE, Academic) do not work with containers.
Please contact your sales representative at [sales@gurobi.com](mailto:sales@gurobi.com) to discuss licensing options. 

## Using the client license

You will need to specify a set of properties to connect to a license server.  You have two options:
* Mounting the client license file:
You can store connection parameters in a client license file (typically called `gurobi.lic`) 
and mount it to the container. 
This option provides a simple approach for testing with Docker.
When using Kubernetes, the license file can be stored as a secret and mounted in the container.

* Setting parameters through environment variables for a WLS license: GRB_WLSACCESSID, GRB_WLSSECRET, and GRB_LICENSEID.
These variables are used to pass the access ID, the secret, and the license ID respectively.

We do not recommend adding the license file to the Docker image itself. It is not a flexible 
solution as you may not reuse the same image with different settings. More importantly, it is not secure
as some license files need to contain credentials in the form of API keys that should remain private.

# How to use this image?

## Using Docker

The following command mounts the license file from the current directory `$PWD` 
and starts a Compute Server instance.

```console
$ docker run  -p61000:61000 \
              --volume=$PWD/gurobi.lic:/opt/gurobi/gurobi.lic:ro  \
              gurobi/compute --hostname=localhost
```

If you have the Gurobi Optimizer client installed locally, you can test your deployment by 
submitting a model for optimization with the ``gurobi_cl`` command line tool.

```
$ gurobi_cl  --server=localhost:61000 ...examples/data/glass4.mps
```

## Using Docker Compose
Example `docker-compose.yml` for a compute server:

```
version: '3.1'
services:
  compute:
    image: gurobi/compute:latest
    restart: always
    ports:
      - "61000:61000"
    command: --hostname=localhost
    volumes:
       - ./gurobi.lic:/opt/gurobi/gurobi.lic:ro
```

Run `$ docker-compose up `

If you have the Gurobi Optimizer client installed locally, you can test your deployment by 
submitting a model for optimization with the ``gurobi_cl`` command line tool.

```
$ gurobi_cl  --server=localhost:61000 ...examples/data/glass4.mps
```

If you want to set up a cluster of Compute Server nodes, we recommend using a 
[Cluster Manager](https://hub.docker.com/r/gurobi/manager).

## Using Kubernetes

If you want to mount a specific license with Kubernetes, it can be done using a secret. 
```
kubectl create secret generic gurobi-lic --from-file="gurobi.lic=$PWD/gurobi.lic"
```

Then you can start a pod that will run the compute server in a container and expose it as a service. 
A simple deployment file is provided as an 
[example](https://github.com/Gurobi/docker-compute/blob/master/9.5.0/k8s.yaml).

```
kubectl apply -f k8s.yaml
```

If you have the Gurobi Optimizer client installed locally, you can test your deployment by 
submitting a model for optimization with the ``gurobi_cl`` command line tool.

```
$ gurobi_cl  --server=localhost:61000 ...examples/data/glass4.mps
```

If you want to set up a cluster of Compute Server nodes, we recommend using a 
[Cluster Manager](https://hub.docker.com/r/gurobi/manager).

# License

By downloading and using this image, you agree with the 
[End-User License Agreement](https://www.gurobi.com/EULA) for the Gurobi software contained in this image.

As with all Docker images, these likely also contain other software which may be under other 
licenses (such as Bash, etc from the base distribution, along with any direct or indirect 
dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use 
of this image complies with any relevant licenses for all software contained within.


