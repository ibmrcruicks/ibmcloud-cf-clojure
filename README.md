# ibmcloud-cf-clojure
Clojure functional programming in Cloud Foundry

Get a simple [Clojure](https://clojure.org) web app running in Cloud Foundry on IBM Cloud.

After watching [Russ Olsen at goto18 on YouTube](https://www.youtube.com/watch?v=0if71HOyVjY), I thought I would go
through the steps needed to start a Clojure app in the Cloud Foundry application runtime environment.

Russ mentioned the [pedestal.io libraries](https://pedestal.io) as a way of bootstrapping a web runtime - this example is based on the [pedestal hello-world sample](https://github.com/pedestal/pedestal/tree/master/samples/hello-world).

## Setup 

### IBM Cloud 
Make sure you have an [IBM Cloud account](https://cloud.ibm.com), and have installed the IBM Cloud [command line tool](https://cloud.ibm.com/docs/cli?topic=cloud-cli-install-ibmcloud-cli).

### Sample code
First, clone the pedestal repo:
```
git clone https://github.com/pedestal/pedestal.git
```
then navigate to the `pedestal/samples/hello-world` directory.

This folder contains a `project.clj` file, which is compatible with the [heroku clojure buildpack](https://github.com/heroku/heroku-buildpack-clojure) -- although the [Cloud Foundry docs](https://github.com/cloudfoundry-community/cf-docs-contrib/wiki/Buildpacks#community-created) point to [mstine's buildpack repo](https://github.com/mstine/heroku-buildpack-clojure), this hasn't been updated since 2013 -- the base heroku repo has moved on, so we'll use that instead.

### Modify service configuration to also support Cloud Foundry
There are two files that make up the sample web app

+ `src/hello-world/service.clj` - settings, routes and config for the app
+ `src/hello-world/server.clj' - the application that listens for 'hello' requests

The default setting for `http/port` in service.clj is 8080 -- this is *usually* the port number passed to Cloud Foundry applications, but it can vary; for completeness, we'll add support for that variability by replacing:
```
  ::http/port   8080
```
with:
```
  ::http/port   (read-string (or (System/getenv "PORT") "8080"))
```
(**read-string** is needed to convert the PORT value from string to number)

Also worth noting that despite what all the Cloud Foundry doc samples suggest, trying to push an application which binds to a TCP port on *localhost* will not work - the server host needs to be unspecified, or set to **"0.0.0.0"**. For that reason, the default bind address for this app needs to be set in the same way:
```
  ::http/host   (or (System/getenv "VCAP_APP_HOST") "0.0.0.0")
```
Save these updates to `src/hello-world/service.clj`, and off we go.

### Deploy to IBM Cloud Foundry

Pick a name for your application, then deploy to IBM Cloud using the "cf push" command. 

We need to specify a couple of parameters:
+ the name of the application
+ the buildpack option, pointing to the Heroku/Cloud Foundry repository
+ the memory size option - set to 256MB to ensure the app will run within the default limit for IBM Cloud Lite accounts (the default for the buildpack is 1GB!)

```
ibmcloud cf push my-first-cf-clojure-app -m256m -b https://github.com/heroku/heroku-buildpack-clojure.git 
```

Run the `cf apps` command to get the fully-qualified hostname for your application; from a browser connect to "**https://**`<your-app-hostname>`**/hello?name=me**", and you should see something like:

```
  Hello me!
```
