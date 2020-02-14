# sMailandStuff

[![Build Status](https://lksk.visualstudio.com/sMailandStuff/_apis/build/status/larskaare.sMailandStuff?branchName=master)](https://lksk.visualstudio.com/sMailandStuff/_build/latest?definitionId=1&branchName=master)
[![Known Vulnerabilities](https://snyk.io/test/github/larskaare/smailandstuff/badge.svg)](https://snyk.io/test/github/larskaare/smailandstuff)

A web application to test various features in web applications, integration to DevOps tools and cloud deployments.

## Features

- oAuth2 and OICD against Azure AD
- Reading email from o365 Graph
- Expose various token related information for user/session
- Exploring cookie behavior for displaying last time visited on a few pages
- Docker multi-stage build
- Support for ESLint
- Security headers like CSP
- Automated tests
- Support for Omnia Radix (CI, CD, Hosting, Monitoring)
- Support for Azure Pipelines, Azure Web App for Containers
- Support for Vulnerability scanning using Snyk

## Install and run locally

- Clone repository
- Do `npm install`
- Create `Application registration` in Azure AD
- Update configuration in `./config/config.js`
- Export the following config into the local environment variables

```bash
export NODE_ENV=development
export CLIENTSECRET=""
export PORT=3000
export TENANTID=""
export CLIENTID=""
```

- Run application using `npm start`

### Test

The the application by running `npm test`

### Linting

Lint the code by running `npm run lint`

### Using nodemon

Using nodemon will automatically restart the application when a change is observed in a javascript file.

To develop using nodemon run `npm run nodemon`

## Docker

- Build: `docker build -t smailandstuff .`
- Run:

```bash
docker run -p 3000:3000  \
    --env TENANTID="" \
    --env CLIENTID="" \
    --env CLIENTSECRET="" \
    smailandstuff
```

## Cloud Deployment

### Radix

Radix lives at <https://www.radix.equinor.com>

An example application may or may not be available at <https://webapp-smailandstuff-production.playground.radix.equinor.com/>.

[Test report from SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=webapp-smailandstuff-production.playground.radix.equinor.com)

[Test report from securityheaders.com](https://securityheaders.com/?q=https%3A%2F%2Fwebapp-smailandstuff-production.playground.radix.equinor.com&followRedirects=on)

- Examine and update `./radixconfig.yaml`
- (Apply for access to the Radix playground)
- Create new application in the Radix Playground
- Inject the CLIENTSECRET using the Radix Web Console

(Current version of code uses memory to as session store. This will not scale beyond one app instance and it will leak memory. This set-up is not recommended for ***real*** production scenarios)

### Azure Pipelines & Azure Web App for Containers

#### Azure Pipelines (CI)

The Azure Pipeline is defined in `azure-pipelines.yml`. The pipeline will build the Docker image the same way Radix does (using Docker multistage build), tag and then push the image to Docker-Hub `larskaare/smailandstuff`. Create your own account on Docker Hub and substitute for `larskaare/smailandstuff` going forward.

#### Azure Web App for Containers (Hosting)

An example application may or may not be available at <https://azsmailandstuff.azurewebsites.net/>.

[Test report from SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=azsmailandstuff.azurewebsites.net)

[Test report from securityheaders.com](https://securityheaders.com/?q=https%3A%2F%2Fazsmailandstuff.azurewebsites.net&followRedirects=on)

To host the application create an Azure Web App For Containers, point it to the `larskaare/smailandstuff` image and the `latest`.

Add the following environment variables to the `Configuration -> Application Settings` section:
- CLIENTID
- CLIENTSECRET
- NODE_ENV
- TENANT_ID

#### Continous Deployment using Docker Hub 

Enable `Continuous Deployment` from the `Container Setting` section of the WebApp (previous section). Copy the webhook to Docker Hub `larskaare/smailandstuff` WebHooks section to enable Docker Hub to trigger a redeploy when new images are pushed into the registry.

### Snyk

https://snyk.io/

The project have defines commands to run a Snyk vulnerability test from the command line. Executing `npm run snyk` will run the test. In order for this to work you will have to do add an environment variable called `SNYK_TOKEN` which holds the your accounts API tokens (There are multiple options for how to authenticate with Snyk)

#### Snyk & Docker

To enable Snyk scanning as part of the Docker Build process, remove the comments from the Snyk section

```Docker
#Running Snyk
ARG SNYK_TOKEN
RUN npm run snyk
```

#### Snyk & Radix

For the SNYK integration to work with Radix you will have to extend the `radixconfig.yaml` to support a builde time secret, the SNYK_TOKEN.

```yaml
sspec:
  build:
    secrets:
      - SNYK_TOKEN
```

The value for the SNYK_TOKEN are entered in the Radix portal (the configuration section of your app.)

#### Snyk & Azure Pipeline

For the SNYK integration to work with Azure Pipelines, running as part of the Docker build, you will have to alter the `azure-pipelines.yml` file to include and arguments parts in the task that builds the Docker image.

```yaml
arguments: --build-arg SNYK_TOKEN=$(SNYKTOKEN)
```

The value for the SNYK_TOKEN is defines as a variable in the Azure Pipelines. Name the varible `SNYKTOKEN`

There are quite a few other ways to integrate Snyk with Azure Pipelines.

__
