# GitHub actions-runner-controller (ARC)

Welcome to the release repo for the [`actions-runner-controller`](https://github.com/actions-runner-controller/actions-runner-controller). This repo contains the official GitHub release of the Action Runner Controller.

`actions-runner-controller` provides GitHub users with the ability to setup auto scaling self-hosted runners for GitHub Actions on a Kubernetes cluster. It provides CRDs (Custom Resource Definition) such as Runner RunnerDeployment HorizontalRunnerAutoscaler, these are used to scale new pods to run your Actions jobs inside.

`actions-runner-controller` is an open-source project currently developed and maintained in collaboration with maintainers @mumoshu and @toast-gear, various [contributors](https://github.com/actions-runner-controller/actions-runner-controller/graphs/contributors), the [awesome community](https://github.com/actions-runner-controller/actions-runner-controller/discussions) and [GitHub](TODO_BLOG_LINK)

## Finding releases

We release all our releases as Helm packages. Helm is a package manager that nakes is easy to install and manage Helm applications and makes managing the installation and updating of ARC far easier.

You can find the Official releases of ARC in the [packages]() of this repo, to find out about supported versions check out our trouble shooting.

## Setup

### 1. Prerequisites

* Install [Helm](https://helm.sh/), the official instructions to setup Helm depending on your platform can be found [here](https://helm.sh/docs/intro/install/)
* [cert-manager](https://cert-manager.io/docs/installation/kubernetes/) is required by the actions-runner-controller, it is used to manage the certificate for the Admission Webhook.

```
# Add the helm repository of the certificate-manager 
$ helm repo add jetstack https://charts.jetstack.io
$ helm repo update

# Install the certificate manager as a chart via helm (note you will need to check what the latest version is)
$ helm install --wait --create-namespace --namespace cert-manager cert-manager jetstack/cert-manager --version v1.8.0 --set installCRDs=true

# Check everything is working properly
$ kubectl --namespace cert-manager get all
```

### 2. Setting up your credentials

There are two ways to setup your credentials for ARC. We recommend using a [GitHub App](https://docs.github.com/en/developers/apps/getting-started-with-apps/about-apps), GitHub Apps allows you to setup a secure integration without having to setup the rotation of PATs.

* Using a GitHub app (not avaliable at Enterprise level)
You can create a GitHub App for either your user account or any organization, below are the app permissions required for each supported type of runner:

_Note: Links are provided further down to create an app for your logged in user account or an organization with the permissions for all runner types set in each link's query string_

**Required Permissions for Repository Runners:**<br />
**Repository Permissions**

* Actions (read)
* Administration (read / write)
* Checks (read) (if you are going to use [Webhook Driven Scaling](#webhook-driven-scaling))
* Metadata (read)

**Required Permissions for Organization Runners:**<br />
**Repository Permissions**

* Actions (read)
* Metadata (read)

**Organization Permissions**
* Self-hosted runners (read / write)

_Note: All API routes mapped to their permissions can be found [here](https://docs.github.com/en/rest/reference/permissions-required-for-github-apps) if you wish to review_

**Subscribe to events**

At this point you have a choice of configuring a webhook, a webhook is needed if you are going to use [webhook driven scaling](#webhook-driven-scaling). The webhook can be configured centrally in the GitHub app itself or separately. In either case the event details are:

* Check run (required for all webhook driven scaling events)
* Workflow job (optionally) (required for [webhook driven scaling with workflow_job events](https://github.com/actions-runner-controller/actions-runner-controller#example-1-scale-on-each-workflow_job-event)

---

**Setup Steps**

If you want to create a GitHub App for your account, open the following link to the creation page, enter any unique name in the "GitHub App name" field, and hit the "Create GitHub App" button at the bottom of the page.

- [Create GitHub Apps on your account](https://github.com/settings/apps/new?url=http://github.com/actions-runner-controller/actions-runner-controller&webhook_active=false&public=false&administration=write&actions=read)

If you want to create a GitHub App for your organization, replace the `:org` part of the following URL with your organization name before opening it. Then enter any unique name in the "GitHub App name" field, and hit the "Create GitHub App" button at the bottom of the page to create a GitHub App.

- [Create GitHub Apps on your organization](https://github.com/organizations/:org/settings/apps/new?url=http://github.com/actions-runner-controller/actions-runner-controller&webhook_active=false&public=false&administration=write&organization_self_hosted_runners=write&actions=read&checks=read)

You will see an *App ID* on the page of the GitHub App you created as follows, the value of this App ID will be used later.

<img width="750" alt="App ID" src="https://user-images.githubusercontent.com/230145/78968802-6e7c8880-7b40-11ea-8b08-0c1b8e6a15f0.png">

Download the private key file by pushing the "Generate a private key" button at the bottom of the GitHub App page. This file will also be used later.

<img width="750" alt="Generate a private key" src="https://user-images.githubusercontent.com/230145/78968805-71777900-7b40-11ea-97e6-55c48dfc44ac.png">

Go to the "Install App" tab on the left side of the page and install the GitHub App that you created for your account or organization.

<img width="750" alt="Install App" src="https://user-images.githubusercontent.com/230145/78968806-72100f80-7b40-11ea-810d-2bd3261e9d40.png">

When the installation is complete, you will be taken to a URL in one of the following formats, the last number of the URL will be used as the Installation ID later (For example, if the URL ends in `settings/installations/12345`, then the Installation ID is `12345`).

- `https://github.com/settings/installations/${INSTALLATION_ID}`
- `https://github.com/organizations/eventreactor/settings/installations/${INSTALLATION_ID}`


Finally, register the App ID (`APP_ID`), Installation ID (`INSTALLATION_ID`), and the downloaded private key file (`PRIVATE_KEY_FILE_PATH`) to Kubernetes as a secret.


* Using a PAT


## Action-runner-controller and GHES



## Troubleshooting

### Getting support

## Contributing

https://github.com/actions-runner-controller/actions-runner-controller#contributing

# FAQs

**Q:** Is everything on the roadmap?

**A:** Almost everything related to features, we might keep the odd feature a surprise if we really aren’t sure about it. We also won’t be tracking all of our bugs here, we have separate repos for these:

# License
This library is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.