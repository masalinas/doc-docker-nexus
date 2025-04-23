# Description
Documentation about Docker Nexus Deployment

Steps to follow 

## STEP 01: Create Nexus Data Volumne

```
$ docker volume create --name nexus-data
```

## STEP 02: Execute Nexus Container

Nexus supports OCI-format Helm charts, so Helm CLI v3.7+ is needed and you must enable OCI support.
These ports must be opened:

- 8081: Nexus Portal
- 5000: Nexus Docker Registry Repository
  
```
$ docker run -d -p 8081:8081 --name gsdpi-nexus -e HELM_EXPERIMENTAL_OCI=1 -v nexus-data:/nexus-data sonatype/nexus3
```

## STEP 03: Recover default admin password

```
$ docker exec consum-nexus cat /nexus-data/admin.password
26884375-0ba2-4c3f-8a6f-07cac8e6586d
```

## STEP 04: Open Nexus and change default admin password

Sign in as admin with the previous default password and complete the wizard to finalize configuration. Open nexus Portal from your browsert

```
http://localhost:8081/
```

Set new admin password

![Nexus password](./images/nexus-new-password.png "Nexus password")

Nexus portal

![Nexus portal](./images/nexus-access.png "Nexus portal")

## STEP 05: Create and configure repositories

Now we will create our own repositories for:

- Docker Images
- Kubernetes charts
- NPM Packages

Go to Configuration -> Repositories -> Add Respository

![Nexus portal](./images/nexus-repository.png "Nexus Repository")

This case it's a Docker repository with this configuration:

- Name: gsdpi-docker
- Format: docker
- Type: hosted
- HTTP: 5000
- Allow Anonymous: false

In case Kubernetes charts:

- Name: gsdpi-helm
- Format: helm
- Type: hosted
- Allow Anonymous: false

In case npm packages

- Name: gsdpi-npm
- Format: npm
- Type: hosted
- Allow Anonymous: false

Now we must reopen docker with these new port 5000 for docker. So we must destroy nexus and re-run again with this command. The last configuration will not be destroyed because is saved in a volume:

```
$ docker run -d -p 8081:8081 -p 5000:5000 --name gsdpi-nexus -e HELM_EXPERIMENTAL_OCI=1 -v nexus-data:/nexus-data sonatype/nexus3
```

## STEP 06: exclude your local nexus repository from https docker login

Our default docker CLI uses https connection to login any docker repository, so we must exclude our local nexus service from docker CLI, editing this file **/etc/docker/daemon.json** and adding our local dns: <MY_LOCAL_PUBLIC/PRIVATE_IP>.nip.io:5000 nexus service like this:

```
$ sudo nano /etc/docker/daemon.json
{
     "insecure-registries" : [ "172.30.0.0/16", "<MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000" ]
}
```

This operation must be execute in all computers in our network where you want access to nexus from http

## STEP 07: creare a rol and user to access to docker nexus repository

Now we are create a role

Go to Nexus configuration -> Roles -> add Role

![Nexus Role](./images/nexus-role.png "Nexus Role")

The configuration for this role is:

- Role Id: gsdpi-developer
- Name: gsdpi-developer
- Description: GSDPI Developer
- Priviledges: nx-repository-view-docker-****-****

Now we are create a user to access to our docker registry

Go to Nexus configuration -> Users -> add User

![Nexus RUseroles](./images/nexus-user.png "Nexus User")

The configuration for this role is:

- Id: <ID_USERNAME>
- First name: <MY_FIRST_NAME>
- Last name: <MY_LAST_NAME>
- Email: <MY_EMAIL>
- roles available: gsdpi-developer

## STEP 08: login in your new nexus repository
Now nexus is running with all repositories created and configured and your local computer is configured to use a http conection and with a local user with priviledges to pull/push images in gsdpi docker repository. Login inside nexus with the


```
$ docker login <MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000 -u <ID_USERNAME>
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credential-stores

Login Succeeded
```
## STEP 09: Push docker image

You must build and tag your image using your nexus <MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io dns address.

```
$ docker build -t <MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000/uniovi-gsdpi-bokeh-epigenomics:0.2 .
The push refers to repository [<MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000/uniovi-gsdpi-bokeh-epigenomics]
188257e1a019: Pushed 
f7cab2190f54: Pushed 
9a9fb8d93b00: Pushed 
e535852bc373: Pushed 
df7e0efe85e6: Pushed 
832ecaad9cb5: Pushed 
7cc0d5f7bb31: Pushed 
ea680fbff095: Pushed 
0.1: digest: sha256:6e415144441b2adf940b8fcceb461616a000e0ca0bd10b0ffaca0d98281fccd5 size: 1999

$ docker push <MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000/uniovi-gsdpi-bokeh-epigenomics:0.2
```

## STEP 10: Pull docker image
Don't forget configure your **/etc/docker/daemon.json** file and include your <MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000 dns to use the protocol http for this repository:

```
$ docker pull <MY_LOCAL_PUBLIC_OR_PRIVATE_IP>.nip.io:5000/uniovi-gsdpi-bokeh-epigenomics:0.2
```

## Links

- [Official Nexus Docker Link](https://hub.docker.com/r/sonatype/nexus3/)
