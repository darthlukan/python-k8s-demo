# python-k8s-demo

Author: Brian Tomlinson <darthlukan@gmail.com>

## Description

A simple python script deployable to openshift for demo purposes.

The purpose of this repo is to show, in the simplest of automated terms, how to get a Python app into a container and
deployed to Kubernetes / OpenShift.

There are, of course, many ways to accomplish this and some are more appropriate depending upon the complexity of the
application, though there are common core principles:

1. The application and its dependencies should be injected into one or more Containerfile(s)/Dockerfile(s) dependent
   upon the application architecture/components.
2. At a minimum deployment to Kubernetes / OpenShift requires the yaml file definition of a `Deployment` or `DeploymentConfig`
   with one or more `spec.template.containers` defined, and a `Namespace` in which to reside on the cluster.
3. One or more Container Images needs to be built from the Containerfile(s)/Dockerfile(s) and then pushed to a registry.
4. The yaml files (manifests) must be applied to the cluster via the equivalent of `kubectl|oc apply -f $FILENAME` by a
   user with appropriate permissions.

More robust applications which require multiple component `Pods` which must communicate with each other will require 
additional manifests for objects such as `Deployments`, `ConfigMaps`, `Services`, `Routes`, `Secrets`, and
`NetworkPolicies`, among others. Refer to the documentation for your Kubernetes distribution or your company's Platform
Team for guidance.


## Implementation

In order to keep things as simple as possible, this repository uses a glorified "Hello, World!"
[Python](https://python.org) script running in an infinite loop to simulate an application.

The script is then copied into the container image via the `COPY` statement in the `Containerfile`, which itself is
based upon a minimal Container Image which provides the Python 3.9 interpreter. Because the script has no dependencies
beyond the Python Standard Library, no `RUN` statements are required to install additional packages.

Within the `deploy` directory at the root of this repo are two files: `deployment.yaml` and `namespace.yaml`. The
namespace is the target where our `Deployment` (`deployment.yaml`) and resulting `Pod` will reside and execute. Because
this "app" is not meant to serve an endpoint nor communicate with other pods, objects such as `Service`, `Route`, and
`NetworkPolicy` are not required.

The image is built via [Podman](https://podman.io) and deployed to a registry via the `make` targets `build` and `push`
respectively. These targets are defined in the `Makefile`. Deploying the configuration manifests is done via the
supplied [Ansible](https://www.ansible.com) playbook named `playbook.yaml`, which can be executed via the `make` target
`deploy` to deploy the application to the cluster, or the target `remove` to remove the application and its namespace
from the cluster. Both of these make targets are provided for convenience to reduce typing.

In order for the playbook to execute, the host system should have the Ansible collection `kubernetes.core` installed via
`ansible-galaxy collection install kubernetes.core`, and either or both of `kubectl` and `oc` installed. The user should
authenticate to their cluster or export the `KUBECONFIG` environment variable BEFORE executing the playbook to ensure
the manifests are applied correctly.


## Q&A

1. Q: "Why did you use Ansible and not [Helm](https://helm.sh/) to deploy the app?"
   A: 12 lines of code in a single playbook.yaml versus 20+ spread across two files in Helm.

2. Q: "Why did you use 'Containerfile' instead of 'Dockerfile'? Do you hate [Docker](https://www.docker.com/)?
   A: 'Containerfile' is more accurate a name and doesn't imply the necessity for a particular tool. Docker doesn't have
   a monopoly on containers and despite what young / uninitiated devs may think, did not invent containers. No, I do not
   hate Docker, far from it, they've contributed quite a lot to the usability of containers over the years and are an
   [OCI](https://opencontainers.org/) founding contributor.

3. Q: "Why didn't you use a more complex Python application/script?"
   A: The purpose of this "demo" is to show just how simple containerization and deployment to Kubernetes / OpenShift
   really is, not to gatekeep behind needless toil. A more robust application would be cool, but the point isn't to wow
   people with a complex application, it's to assist would-be container application developers and operators in
   understanding the fundamentals so that they can grow.

4. Q: "Why do you execute `ansible-playbook` via `make`?
   A: To reduce typing so as not to distract from what the automation is doing. Also, `make` is a great, reliable tool.

5. Q: "Why did you use `registry.access.redhat.com/ubi8/python-39` instead of `alpine` or `$INSERT_IMAGE_HERE`?
   A: Again, so as not to distract from the core concepts, namely, getting an app into a container image. I needed an 
   image which had Python loaded, `ubi8/python-39` comes with Python 3.9. The resulting `Containerfile` is 3 source 
   lines of code. Win-win.

6. Q: "Where can I learn more about container application development?"
   A: [This Red Hat Developers](https://developers.redhat.com/getting-started) page has lots of useful references, as 
   does [this Podman documentation](https://docs.podman.io/en/latest/Introduction.html).
   [Toolbox](https://containertoolbx.org/) is also a useful tool for getting started with using containers as
   development environments.
