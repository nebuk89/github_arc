### 1.B Using a PAT
Login to a GitHub account which has admin privileges for the repo you require and go to https://github.com/settings/tokens. Click on the new token button (this is to generate a new [PAT](https://github.com/settings/tokens?type=beta))

<img width="750" alt="App ID" src="/Users/nebuk89/Documents/GitHub/github_arc/PAT.png">

You will then need to set the right scopes for the PAT depending what level you are setting up your runners at:

**Required Scopes for Repository Runners**

* repo (Full control)

**Required Scopes for Organization Runners**

* repo (Full control)
* admin:org (Full control)
* admin:public_key (read:public_key)
* admin:repo_hook (read:repo_hook)
* admin:org_hook (Full control)
* notifications (Full control)
* workflow (Full control)

**Required Scopes for Enterprise Runners**

* admin:enterprise (manage_runners:enterprise)


Once you have chosen your scope, hit create and take a note of the token that has been generated for you, we are going to use it in our next step.

### 2.B Custom Values YAML for PATs

You will now need to take the token that we just generated and add it to a new file called `custom-values.yaml` and add the following contents to it:  

```
authSecret:
  github_token: REPLACE_YOUR_TOKEN_HERE
  create: true
```

You can add more later, to find out what is supported in the `custom-values.yaml` check out our docs here.