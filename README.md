# Manifest to my current local minikube setup 

After some interesting and rather successful experiences running Kubernetes in Docker Desktop **kind** (though also sometimes a bit painful - lastly I spent hours without success to upgrade my Kubernetes-version), the time had finally come to switch to a **minikube** cluster running on my local ARM64 MacBook with M1-processor. 

So around June 12, 2023 I installed minikube, read some pages and had the analogous service cluster setup as in DD up and running in no time. Delightful! 

I started to deploy some simple toy Spring Boot apps - partially as native (GraalVM) image, partially classical ARM64 images on VM. Quickly I needed persistent volumes, as one of the apps uses a file H2-database.
Others run on Postgres in Docker, resp. I migrated them there. So the next step was to run a Postgres-Database inside the cluster with postgres-service exposing it internally to the Spring apps.
The app deployments are exposed by services of type LoadBalancer, which are easily made accesible in minikube using tunnels.

Next step was to externalize the Spring configurations via ConfigMaps and introducing a secret for the credentials to Postgres. Finally I wanted to be able to observe and manage the database, so I installed a PGAdmin as StatefulSet, together with a PVC and a pgadmin-service that minikube can on-demand expose to my browser.

Further Ideas, to come up, I hope: 
- Oauth2 security with new Spring 3.1.x Oauth2-Authorization-Server
- Service registry with spring cloud
- Spring Cloud Gateway and most probably Spring Cloud Configuration Server
- Some toy use-cases for Spring Cloud Kubernetes (maybe)
- Migrate more apps - my Webflux Account service, one(s) serving a Vue- or React-Frontend, maybe some Python-based apps
- Functionality for Scaling-To-Zero with KNative (should be very useful with growing landscape)
- Playground for ArgoCD, service mesh as Istio, etc.

## Repository Contents

> The yaml files for orchestrating the services landscape described above. A subfolder `pvc` comprises all PersistentVolumeClaims, such that I can remove all deployments and services from the root-directory with the `kubectl delete -f .`command and restart it with `kubectl apply -f .` without losing the persistent data in the PVs.   

> Few remainders of the previous kind setup, mostly consisting of a ClusterRoleBinding and ServiceAccount in the `kind`-subfolder,


## Progress

12.06.23 Initial minikue setup with hello-world service.

19.06.23 Five services are running:
- a **quiz-engine** service (Basic Auth) with an extensible Web-Quiz as CRUD-Rest Spring Boot application on Json-Basis (no frontend).
- a **code-sharing** service (Basic auth) - à la pastebin - to store and exchange code snippets (rudimentary Thymeleaf-rendered HTML frontend and Json-API).
- a **recipe** service - running as Native image on Spring Boot 3.1.x, where recipes can be stored, patched and queried
- a postgres-service using a Postgres-OCI on alpine basis, running multiple databases (two as of today)
- a pgadmin-service to manage the database via NodePort exposure from the browser.

03.07.23 Added deployment for authorization-gateway, consisting of a shared pod with
- a Spring Cloud gateway container, whose (native image) app also implements an oauth2-client, that connects via authorization_code workflow to the Spring authorization server (see below). Also it offers a webflux endpoint for user registry, that persists into the cloud Postgresql-database "users".
- a Spring Boot container running a Spring oauth2-authorization-server, that is addressed by the gateway and uses the users-database to authorize OIDC logins, redirected by the gateway. It serves id- and JWT-tokens for accessing the Oauth2-ressourceServers included in the Spring Boot Web-services.