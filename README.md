# license-finder
This repository contains the workflow and config files for the license-finder tool.

It uses a customized image for https://github.com/pivotal/LicenseFinder

This image is used to scan projects for dependencies 
and  checks the licenses against a whitelist to ensure compliance.

# How to use the workflow
The `.github/workflow/license_finder.yml` workflow can be invoked by other repositories.

Here the steps to follow:

1. Copy the `invoke_license_finder.yml` workflow to the `.github/workflows` folder of your project repo

2. Go to the Actions tab of your project, select "`invoke_license_finder`" from the list and run it


# Adding licences / manage configuration

### Background

It **will** often happen that the licence_finder won't "approve" the licenses of the dependencies, since these dependencies do not have properly configured licenses-files in them.  In this case you have to 

1. identify the dependencies and their licenses

2. reach out to our Legal-department and get their ***written*** & ***documented*** approval for the specified licenses

***So, before you apply any changes to the configurations - be sure to have done that in very first place! -> Run the scanner against your local project and identity the licenses & reach out to Legal.***



### The practical part

>  *As of December 2024, we have only once central "dependency_decisions.yml" which keeps **all** the licenses and dependencies. This is subject to change in the scope of https://mzerolabs.atlassian.net/browse/CL-3293.*


Configuration of the image is done via adjustments of the

```
doc/dependency_decisions.yml
```
file in **this** projects directory. If your new project build is failing because the licenses are not permitted, adjust the ```doc/dependency_decisions.yml```.

#### Adjust & test the configuration with your project's specific "findings"

To add and test your changes **locally**, **before** creating a merge-request here, do following:

1. having the list of the depencencies from a first local run 
```
docker run -ti -v "$(pwd):/$(basename "$(pwd)")" -w "/$(basename "$(pwd)")" --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder --prepare --composer-check-require-only=true --decisions-file=/doc/dependency_decisions.yml --python-version=3"
```

#### Add new licenses/dependendies

Copy the `doc/dependency_decisions.yml`from the license-finder repository to your projects directory
```
cp license-finder/doc/dependency_decisions.yml ttg-frontend/
```
enter your projects directory, and then, **per dependency** - run "license_finder approvals add" to add that new dependency:

```
docker run -ti -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder approvals add '@csstools/selector-resolve-nested' --why='some reason' --who='Vladimir Vecgailis'"
```
If there are some licences to be approved - run "license_finder permitted_licenses add" **per license** to add that new license:

```
docker run -ti -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder permitted_licenses add "GPL-5" --why='some licence approval reason' --who='Vladimir Vecgailis'"
```
#### Verify your "decisions"
Now, the file `dependency_decisions.yml` in your directory was modified.
To verify whether the licence_finder will now properly handle all your approvals, re-run the license-finder with your local  `dependency_decisions.yml` mounted:

```
docker run -ti -v "$(pwd):/$(basename "$(pwd)")" -v $(pwd)/dependency_decisions.yml:/doc/dependency_decisions.yml -w "/$(basename "$(pwd)")" --entrypoint /bin/bash registry.gitlab.com/crosslend/docker/license-finder/main:latest -c "source /usr/share/rvm/scripts/rvm; license_finder --prepare --composer-check-require-only=true --decisions-file=/doc/dependency_decisions.yml --python-version=3"
```

#### Persist your "decisions"

Final part - add your changes to the license-finder repository

- copy back the altered dependency_decisions.yml to https://gitlab.com/crosslend/docker/license-finder/-/blob/main/doc/dependency_decisions.yml
- create a MR and merge it
Upon a next build your projects "new" licenses/dependencies will be taken into account




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

