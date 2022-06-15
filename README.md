# GitHub actions-runner-controller (ARC)

Welcome to the release repo for the [`actions-runner-controller`](https://github.com/actions-runner-controller/actions-runner-controller). This repo contains the official GitHub release of the Action Runner Controller.

`actions-runner-controller` provides GitHub users with the ability to setup auto scaling self-hosted runners for GitHub Actions on a Kubernetes cluster. It provides CRDs (Custom Resource Definition) such as Runner RunnerDeployment HorizontalRunnerAutoscaler, these are used to scale new pods to run your Actions jobs inside.

`actions-runner-controller` is an open-source project currently developed and maintained in collaboration with maintainers @mumoshu and @toast-gear, various [contributors](https://github.com/actions-runner-controller/actions-runner-controller/graphs/contributors), the [awesome community](https://github.com/actions-runner-controller/actions-runner-controller/discussions) and [GitHub](TODO_BLOG_LINK)

## Finding releases

We release all our releases as Helm charts. Helm is a package manager that nakes is easy to install and manage Helm applications and makes managing the installation and updating of ARC far easier.

You can find the Official releases of ARC in the [packages]() of this repo, to find out about supported versions check out our trouble shooting.

## Prerequisites

This guide assumes you already have access to a Kubernetes instance which is in target for your current context.

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

---

## Setup

Your only need to do 1a or 1b. Please use 1a unless setting up runners at an Enterprise Level.

### 1.A Setting up your credentials - GitHub App

There are two ways to setup your credentials for ARC. We recommend using a [GitHub App](https://docs.github.com/en/developers/apps/getting-started-with-apps/about-apps), GitHub Apps allows you to setup a secure integration without having to setup the rotation of PATs.

Using a GitHub app (not avaliable at Enterprise level)
You can create a GitHub App for either your user account or any organization, below are the app permissions required for each supported type of runner:

_Note: Links are provided further down to create an app for your logged in user account or an organization with the permissions for all runner types set in each link's query string_

| _Prem Level_      | Repository Runners            | Organization Runners               |
|-------------------|-------------------------------|------------------------------------|
| _Repo Perm_       | Actions (read)                | Actions (read)                     |
| _Repo Perm_       | Administration (read / write) |                                    |
| _Repo Perm_       | Checks (read)                 |                                    |
| _Repo Perm_       | Metadata (read)               | Metadata (read)                    |
|                   |                               |                                    |
| _Enterprise Perm_ |                               | Self-hosted runners (read / write) |

_Note: All API routes mapped to their permissions can be found [here](https://docs.github.com/en/rest/reference/permissions-required-for-github-apps)_

**Setup Steps**

To get started for a personal account, open up the [Create GitHub Apps on your account](https://github.com/settings/apps/new?url=http://github.com/actions-runner-controller/actions-runner-controller&webhook_active=false&public=false&administration=write&actions=read), add a unique name and hit the "Create app" button at the botto of the page.

OR

If you want to create a GitHub App for your organization, replace the `:org` part of the following URL with your organization name before opening it. Then enter any unique name in the "GitHub App name" field, and hit the "Create GitHub App" button at the bottom of the page to create a GitHub App.

<https://github.com/organizations/>_YOUR_ORG_NAME_/settings/apps/new?url=<http://github.com/actions-runner-controller/actions-runner-controller&webhook_active=false&public=false&administration=write&organization_self_hosted_runners=write&actions=read&checks=read>

You will see an _App ID_ on the page of the GitHub App you created as follows, **you will want to make a note of this App ID** as we will use it later.

<img width="750" alt="App ID" src="https://user-images.githubusercontent.com/230145/78968802-6e7c8880-7b40-11ea-8b08-0c1b8e6a15f0.png">

Download the private key file by pushing the "Generate a private key" button at the bottom of the GitHub App page. **We will use this file later.**

<img width="750" alt="Generate a private key" src="https://user-images.githubusercontent.com/230145/78968805-71777900-7b40-11ea-97e6-55c48dfc44ac.png">

Go to the "Install App" tab on the left side of the page and install the GitHub App that you created for your account or organization.

<img width="750" alt="Install App" src="https://user-images.githubusercontent.com/230145/78968806-72100f80-7b40-11ea-810d-2bd3261e9d40.png">

When the installation is complete, you will be taken to a URL in one of the following formats, the last number of the URL will be used as the Installation ID later (For example, if the URL ends in `settings/installations/12345`, then the Installation ID is `12345`).

* `https://github.com/settings/installations/${INSTALLATION_ID}`
* `https://github.com/organizations/eventreactor/settings/installations/${INSTALLATION_ID}`

You shoudl now have an:
* (`APP_ID`)
* Installation ID (`INSTALLATION_ID`)
* Private key file (`PRIVATE_KEY_FILE_PATH`)

### [1.B Using a PAT](using_pats.md)

[See the PAT readme](using_pats.md)

### 2 Adding credentials to your custom-values.yaml

You will need to create a new file called `custom-values.yaml`, this is what we will use to setup our credentials for our runner. Once you have the file update the contents to use the credentials we got in the previous step.

```
authSecret:
  github_app_id: REPLACE_APP_ID
  github_app_installation_id: REPLACE_INSTALLATION_ID
  github_app_private_key: REPLACE_PATH_TO_DOWNLOADED_PRIVATE_KEY
  create: true
```

### 3 Install action-runner-controller using Helm

You are now ready to setup ARC using your new values file!

```
# Add the Helm repository
$ helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
# Install the Helm Chartchart
$ helm install -f custom-values.yaml --wait --namespace actions-runner-cont --create-namespace actions-runner-controller actions-runner-controller/actions-runner-controller
# Verify installation
$ kubectl --namespace actions-runner-cont get all
```

### 4 Deploy your runner

We now have the controller deployed on our K8s instance but have no place for our runners to exist nor a first runner for GitHub to reference.

First we will create a namespace for our runners to live in.

```
kubectl create namespace arc-scalable-runners
```

Next we will need to create another file called `runner-scale.yaml` which will contain the details of how we want to scale those runners. Make sure you update either your repo name you want to add the runners to or the Org name as the namespace in this file!

This is done by swapping the key word for where you are targetting: 
  repository:
  organization: 
  enterprise:
```
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: runner-deployment
spec:
  template:
    spec:
      organization: your-organization-name
---
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: runner-deployment-autoscaler
spec:
  scaleTargetRef:
    name: runner-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
    organization:
    - your-organization-name
```

With this file updated with your details, we are ready to create our runners!

```
kubectl --namespace arc-scalable-runners apply -f runner-scale.yaml
```

We will now check in Kubernetes that we have our first runner.

```
kubectl --namespace arc-scalable-runners get runner
```

Last we will want to now go to GitHub and check that we can seet his runner

### Next steps

## Action-runner-controller and GHES

## Troubleshooting

### Getting support

## Contributing

<https://github.com/actions-runner-controller/actions-runner-controller#contributing>

# FAQs

**Q:** Is everything on the roadmap?

**A:** Almost everything related to features, we might keep the odd feature a surprise if we really aren’t sure about it. We also won’t be tracking all of our bugs here, we have separate repos for these:

# License

This library is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License.
