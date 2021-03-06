# rfiweb

`rfiweb` is a flask front-end for Andy Yuen's excellent [RCM demo](https://github.com/AndyYuen/rcm2)

## Deployment

`rfiweb` can be deployed standalone or to OpenShift.

### standalone

Install the requirements, preferably in a virtual environment:
```
pip install -r requirements.txt
```
Start the application with gunicorn:
```
gunicorn wsgi:application
```

### podman and containers

An `rfiweb` container image is available from [quay.io](https://quay.io/smileyfritz/rfiweb). This container image doesn't allow for Keycloak authorization. You can run the container with `podman` or `docker` locally:
```
podman run -d --name=rfiweb --net=host \
-e RHPAM_USER=user \
-e RHPAM_PASS="password" \
-e RHPAM_URI="http://kie-server-url-without-trailing-slashes" \
-e CONTAINER_ID=RCM2_1.0.1 \
-e FLASK_SECRET=1234 \
-ti quay.io/smileyfritz/rfiweb:latest
```
Once the container creates, you can verify with `podman ps`:
```
$ podman ps
CONTAINER ID  IMAGE                              COMMAND               CREATED        STATUS            PORTS  NAMES
73c4d7e1a83b  quay.io/smileyfritz/rfiweb:latest  gunicorn wsgi:app...  3 seconds ago  Up 3 seconds ago         rfiweb
```
The application will now be available at `http://localhost:8000`

### openshift

Create a new application from the python 3.6 s2i builder:
```
oc new-app python:3.6~https://github.com/shaneboulden/rfiweb.git
oc expose svc/rfiweb
```
Set the following environment variables in the deployment config:
```
RHPAM_URI    - the base Process Automation Manager (PAM) URI
RHPAM_USER   - the PAM user
RHPAM_PASS   - the PAM pass
CONTAINER_ID - the ID of the deployed PAM container, eg; RCM2_1.0.1
FLASK_SECRET - a value for CSRF protection
```

## keycloak configuration

RFIweb can be configured with Keycloak/Red Hat Single Sign On.

Firstly, create a new SSO container instance ensuring that the admin username and password are set. The default routes will be created with a TLS re-encrypt policy, which is fine.

Create an RFI web instance:
```
oc new-app python:3.6~https://github.com/jockey10/rfiweb.git
``` 
Create a new secure route for RFIweb using a TLS edge policy.

Create a client in SSO with the HTTPS URL for RFIweb. Ensure a user is created for the application.

Create a file `keycloak.json` in the `static` directory of rfiweb, and add the following:
```
{
  "realm" : "keycloak realm for rfiweb",
  "auth-server-url" : "https://sso-server/auth",
  "resource" : "keycloak client id for rfiweb",
  "public-client": "true"
}
```
Access the RFIweb URL from OpenShift, and you should be redirected to Keycloak.
