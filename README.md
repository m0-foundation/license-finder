# license-finder
This repository contains a GitHub Actions workflow template and configuration for **license-finder**, a tool for scanning projects to validate software licenses in the code and its dependencies.

Validation is performed against a whitelist to ensure compliance.

The Docker image for scanning is built and managed in https://gitlab.com/crosslend/docker/license-finder

The configuration file [dependency_decisions.yml](https://github.com/m0-foundation/license-finder/blob/main/doc/dependency_decisions.yml) includes licenses and projects approved by The Thing GmbH's legal department (legal@m0.xyz).
Please reach out to legal to approve new licenses & dependencies.



# How to use the workflow
The `.github/workflows/license_finder.yml` workflow can be invoked by other repositories.

#### Steps:

1. Copy the `invoke_license_finder.yml` file into the `.github/workflows` folder of your project repository, commit and push it.
2. In your repository's **Actions** tab, select `invoke_license_finder` from the list and run it.
3. The workflow will pull the Docker image from https://gitlab.com/crosslend/docker/license-finder and use the [dependency_decisions.yml](https://github.com/m0-foundation/license-finder/blob/main/doc/dependency_decisions.yml) file from this repository as its configuration. A report will be generated for you to review.



# Adding new licences / dependencies

### Background

It **will** often happen that the **license_finder** won't "approve" the licenses of certain dependencies because their license files are not properly configured, missing or not yet approved. In such cases, you **must**:

1. Identify the dependencies and their associated licenses.
2. Reach out to the Legal department and obtain their ***written*** and ***documented*** approval for the specified licenses.

***Before applying any changes to the configurations, ensure you have completed this process! Run the scanner locally on your project to identify the licenses, and reach out to Legal.***

>  *As of December 2024, we have only once central "dependency_decisions.yml" which keeps **all** the licenses and dependencies. This is subject to change in the scope of https://mzerolabs.atlassian.net/browse/CL-3293, after which also project-specific "dependency_decisions"-files will be possible*


Configuration of the license-finder image is done via adjustments of the

```
doc/dependency_decisions.yml
```
file in **this** projects directory. If your new project build is displaying that the licenses are not permitted, adjust the ```doc/dependency_decisions.yml```.

#### Add new licenses/dependencies

Copy the `doc/dependency_decisions.yml`from the license-finder repository to your projects directory
```
cp license-finder/doc/dependency_decisions.yml ttg-frontend/
```
enter your projects directory, and then, **per dependency** - run "license_finder approvals add" to add that new dependency:

Example(please adjust for your case):

```
docker run -ti -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder approvals add '@csstools/selector-resolve-nested' --why='some reason' --who='Vladimir Vecgailis'"
```
If there are some licenses to be approved - run "license_finder permitted_licenses add" **per license** to add that new license:

Example(please adjust for your case):

```
docker run -ti -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder permitted_licenses add "GPL-5" --why='some licence approval reason' --who='Vladimir Vecgailis'"
```
#### Verify your "decisions"
Now, the file `dependency_decisions.yml` in your projects directory was modified.
To verify whether the license-finder will now properly handle all your approvals, re-run the license-finder with your local  `dependency_decisions.yml` as configuration:

```
docker run -ti -v "$(pwd):/$(basename "$(pwd)")" -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml -w "/$(basename "$(pwd)")" --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder --prepare --composer-check-require-only=true --decisions-file=/doc/dependency_decisions.yml --python-version=3"
```

#### Persist your "decisions"

Final part - add your changes to the license-finder repository

- copy back the altered dependency_decisions.yml to https://gitlab.com/crosslend/docker/license-finder/-/blob/main/doc/dependency_decisions.yml
- create a MR and merge it
Upon a next build your projects "new" licenses/dependencies will be taken into account



## Test any project locally

To test **any** random project for the licenses & dependencies used do following:

1. Clone the project in question
   1. git clone https://github.com/foo/myproject/
   2. cd myproproject
2. Copy the configuration file [dependency_decisions.yml](https://github.com/m0-foundation/license-finder/blob/main/doc/dependency_decisions.yml)  to that projects folder
3. Run

```
docker run -ti -v "$(pwd):/$(basename "$(pwd)")" -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml -w "/$(basename "$(pwd)")" --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder --prepare --composer-check-require-only=true --decisions-file=/doc/dependency_decisions.yml --python-version=3"
```

This will scan the project "myproject" using the `doc/dependency_decisions.yml` configuration file from this repository here.

## Other infos

1. Use the helper `./license` to access the license_manager binary in the docker image and manage the decision file
   - For use with python projects containing dependencies to the CrossLend
     [PyPi repository](https://gitlab.com/crosslend/devops/pypi), please make sure to provide a valid Index URL
     including GitLab token (either via `--extra-index-url ...` setting in the requirements.txt or environment variable
     `PIP_EXTRA_INDEX_URL`).



## Useful commands

View permitted licenses:
   `./license_finder permitted_licenses list`

Add permitted license: 
   `./license_finder permitted_licenses add LICENSE [--who=WHO] [--why=WHY]`

Add allowed dependency
   `./license_finder approvals add DEPENDENCY [--who=WHO] [--why=WHY]`

