---
tags: [GithubActions Demo Script]
title: 08 - Custom Action
created: '2020-06-27T09:47:36.112Z'
modified: '2020-06-27T10:00:38.708Z'
---

# 08 - Custom Action

## Intro to Action
1. Two types of actions:
  a. [Docker Container Actions](https://help.github.com/en/actions/creating-actions/creating-a-docker-container-action)
    - Can include any dependencies
    - Works only on linux runners/agents  
    - Can be from `Dockerfile` based or from "Docker Hub"
    - Comparatively slower to run

    b. [Javascript Actions](https://help.github.com/en/actions/creating-actions/creating-a-javascript-action)  
      - Use GitHub actions npm packages
      - Runs on the agent machine (windows or linux)
      - Cannot assume dependencies
      - Comparatively faster to run
2. Can be hosted in either:
  - A GitHub public repo
  - Within the same private repo as the workflow
  - **Note**: _Not from other private repos, even in the same organisation_

## Custom Javascript Action
3. Sample Action that scans for secrets in config files.
4. Clone an empty public repo. E.g. `napalicorp/cred-scanner`
5. Add `Action.yml` with below:
```
name: 'Cred Scanner'
description: 'Finds credentials in source files'
inputs:
  pathToSearch:
    description: 'Path to search for files'
    required: true
    default: '.'
  fileTypes:
    description: 'File types to search for'
    required: true
    default: '.cs$'
outputs:
  foundSecrets:
    description: 'true if found secrents, otherwise false'
runs:
  using: 'node12'
  main: 'index.js'
```
6. `npm init -y` on the repo to make it a nodejs project
7. Install GitHub actions tools:
  - `npm install @actions/core`
  - `npm install @actions/github`
7. Add `index.js` with below:
```
const core = require('@actions/core');
const github = require('@actions/github');
const path = require('path');
const fs = require('fs');

try {
    console.log("Starting credentials scanner");
    const pathToSearch = core.getInput('pathToSearch');
    console.log(`Path to search=${pathToSearch}`);
    const fileTypes = core.getInput('fileTypes');
    console.log(`Fle types=${fileTypes}`);
    const payload = JSON.stringify(github.context.payload, undefined, 2);
    console.log(`Payload: ${payload}`);

    var found = findSecrets(pathToSearch, fileTypes);
    if(found && found.length > 0){
        core.setFailed('Found secrets in source files!');
        core.setOutput('foundSecrets', true);
    }else{
        console.log("No secrets found in the files");
        core.setOutput('foundSecrets', false);
    }
} catch (error) {
    core.setFailed(error.message);
}

function findSecrets(dir, fileExtension){
    let files = [];
    fs.readdirSync(dir).forEach(file => {
        const filePath = path.join(dir, file);
        const stat = fs.lstatSync(filePath);

        // If we hit a directory, apply our function to that dir. If we hit a file, check if it contains a secret
        if (stat.isDirectory()) {
            const filesWithSecrets = findSecrets(filePath, fileExtension);
            files = files.concat(filesWithSecrets);
        } else {
            if (path.extname(file) === fileExtension) {
                const fileContent = fs.readFileSync(filePath);
                const regex = new RegExp("(\"|')[a-z0-9\/+]{40}(\"|')", "ig");
                if (regex.test(fileContent)) {
                    console.log(`A secret was found in the file: ${filePath}`);
                    files.push(filePath);
                }
            }
        }
    });

    return files;
}
```
8. Push the all of the files including `node_modules` to Github repo.


[< Back to TOC](README.md)