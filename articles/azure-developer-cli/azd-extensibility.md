---
title: Customize your Azure Developer CLI workflows using command and event hooks
description: Explores how to use Azure Developer CLI hooks to customize deployment pipelines
author: alexwolfmsft
ms.author: alexwolf
ms.date: 1/27/2023
ms.topic: reference
ms.custom: devx-track-azdevcli
ms.service: azure-dev-cli
---

# Customize your Azure Developer CLI workflows using command and event hooks

The Azure Developer CLI supports various extension points to customize your workflows and deployments. The hooks middleware allows you to execute custom scripts before and after `azd` commands and service lifecycle events. hooks follow a naming convention using *pre* and *post* prefixes on the matching `azd` command or service event name. 

For example, you may want to run a custom script in the following scenarios:

* Use the *prerestore* hook to customize dependency management.
* Use the *predeploy* hook to verify external dependencies or custom configurations are in place before deploying your app.
* Use the *postup* hook at the end of a workflow or pipeline to perform custom cleanup or logging.

## Available hooks

The following `azd` command hooks are available:

* `prerestore` and `postrestore`: Run before and after package dependencies are restored.
* `preprovision` and `postprovision`: Run before and after Azure resources are created.
* `preinfracreate` and `preinfracrate`: Run before and after Azure resources are created. These values are interchangeable aliases for `preprovision` and `postprovision`.
* `predeploy` and `postdeploy`: Run before and after the application code is deployed to Azure.
* `preup` and `postup`: Run before and after the combined deployment pipeline. `Up` is a shorthand command that runs `restore`, `provision`, and `deploy` sequentially.
* `predown` and `postdown`: Run before and after the resources are removed.
* `preinfradelete` and `preinfradelete`: Run before and after Azure resources are created. These values are interchangeable aliases for `preprovision` and `postprovision`.

The following service lifecycle event hooks are available:

* `prerestore` and `postrestore`: Run before and after the service packages and dependencies are restored.
* `prepackage` and `postpackage`: Run before and after the app is packaged for deployment.
* `predeploy` and `postdeploy`: Run before and after the service code is deployed to Azure.

## Hook configuration

Hooks can be registered in your `azure.yaml` file at the root or within a specific service configuration. All types of hooks support the following configuration options:

* `shell`: `sh` | `pwsh` (automatically inferred from run if not specified).
* `run`: Define an inline script or a path to a file.
* `continueOnError`: When set will continue to execute even after a script error occurred during a command hook (default false).
* `interactive`: When set will bind the running script to the console `stdin`, `stdout` & `stderr` (default false). This property need to be set to `true` in order to see the hook's output.
* `windows`: Specifies that the nested configurations will only apply on windows OS. If this configuration option is excluded, the hook executes on all platforms.
* `posix`: Specifies that the nested configurations will only apply to POSIX based OSes (Linux & MaxOS). If this configuration option is excluded, the hook executes on all platforms.

## Hook examples

The following examples demonstrate different types of hook registrations and configurations.

### Root command registration

Hooks can be configured to run for specific `azd` commands at the root of your `azure.yaml` file.

```yml
name: todo-nodejs-mongo
metadata:
  template: todo-nodejs-mongo@0.0.1-beta
hooks:
  prerestore: # Example of an inline script. (shell is required for inline scripts)
    shell: sh
    run: echo 'Hello'
  preprovision: # Example of external script (Relative path from project root)
    run: ./hooks/preprovision.sh
services:
  web:
    project: ./src/web
    dist: build
    language: js
    host: appservice
  api:
    project: ./src/api
    language: js
    host: appservice
```

### Service registration

Hooks can also be configured to run only for specific services defined in your `.yaml` file.

```yml
name: todo-nodejs-mongo
metadata:
  template: todo-nodejs-mongo@0.0.1-beta
services:
  web:
    project: ./src/web
    dist: build
    language: js
    host: appservice
  api:
    project: ./src/api
    language: js
    host: appservice
    hooks:
      prerestore: # Example of an inline script. (shell is required for inline scripts)
        shell: sh
        run: echo 'Restoring API service...'
      prepackage: # Example of external script (Relative path from service path)
        run: ./hooks/prepackage.sh
```

### OS specific hooks

Optionally, hooks can also be configured to run either on Windows or Posix (Linux & MaxOS). By default, if the Windows or Posix configurations are excluded the hook executes on all platforms.

```yml
name: todo-nodejs-mongo
metadata:
  template: todo-nodejs-mongo@0.0.1-beta
hooks:
  prerestore: 
    posix: # Only runs on Posix environments
      shell: sh
      run: echo 'Hello'
   windows: # Only runs on Windows environments
     shell: pwsh
     run: Write-Host "Hello"
services:
  web:
    project: ./src/web
    dist: build
    language: js
    host: appservice
  api:
    project: ./src/api
    language: js
    host: appservice
```
