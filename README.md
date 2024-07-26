# Description
Documentation about Docker Nexus Deployment

Steps to follow 

## STEP 01: Create Nexus Data Volumne

```
$ docker volume create --name nexus-data
```

## STEP 02: Execute Nexus Container

```
$ docker run -d -p 8081:8081 --name consum-nexus -v nexus-data:/nexus-data sonatype/nexus3
```

## STEP 03: Recover default admin password

```
$ docker exec consum-nexus cat /nexus-data/admin.password
26884375-0ba2-4c3f-8a6f-07cac8e6586d
```

## STEP 04: Open Nexus and change default admin password

Sign in as admin with the previous default password and complete the wizard to finalize configuration

```
GET http://localhost:8081/
```

Set new admin password

![Nexus password](./images/nexus-new-password.png "Nexus password")

Disable access

![Nexus access](./images/nexus-access.png "Nexus access")

## Links

- [Official Nexus Docker Link](https://hub.docker.com/r/sonatype/nexus3/)
