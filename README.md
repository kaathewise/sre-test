# sre-test

## Goal

Create a sample deployment with 2 environments: `dev`, automatically built from
master, and `prod` with controlled release.

### Sub-goals

* Minimize possibility for mistakes
* Minimize possibility for broken code in `prod`
* Provide easy management of scalability and load balancing
* Make operations such as deployment to `prod`, continuous `dev` build, rolling
back to a previous release as effortless as possible
* Ensuring that answering questions like "which code version is released to which
environment?" is as easy as possible

## Solution

### Code structure

The code is split into 2 repositories: this one contains the deployment configurations
and [https://github.com/kaathewise/birthday-app] contains the sample application.

The reasons for such split are the following: these two parts are largely independent,
they may be managed by different people, and their changes should be processed differently.
The split simplifies the management of the project and protects from accidental changes.

### Deployment structure

The deployment is hosted on Google Cloud at [https://console.cloud.google.com/home/dashboard?project=sre-test-203806], and consists
of several services managed by Kubernetes: `prod` service, `dev` service and Jenkins subcluster
responsible for building images and release. We also use Google Container Registry for hosting
Docker containers and CloudSql for app data storage.

### Usage

The code repository has 2 main branches, `master` and `release`.

When the code is submitted to `master`
1. The commit is picked up by Jenkins.
2. The tests are run on this commit.
3. A Docker image is created, annotated with `latest` and a *short hash of the commit*,
and uploaded to GCR.
4. The image is released to the rolling `dev` environment.

When the code is submitted to `release` (it should only be a ff-merge of `master`)
1. The commit is picked up by Jenkins.
2. [In the rare case when it's not actually ff-merge, a new image is created and
uploaded to GCR].
3. The image corresponding to the commit is annotated with `RCyyyyMMdd_HHmm` and `live`.
4. The image is released to the `prod` environment.

It is important to note that
* No change of the deployment configuration is needed when updating the binary
due to the usage of `latest` and `live` tags.
* All images released to `prod` are marked with permanent `RCxxx` tags that makes
finding them easy afterwards.
* Looking either at the GCR tags or at the code repo should be enough to deduce
which version of code is running in each environment.
* By the magic of k8s all updates are happening via rollouts, i.e. with zero downtime.

Releasing a particular dev version to prod can look like this:
```
git pull
git checkout release
git merge fc39658
git push
```

Rolling back is a bit harder:
```
git pull
git branch -f release fc39658
git push origin release --force
```

## Not implemented

### Necessary improvements

1. Separate MySql instance for `dev` environment. Currently each environment has
its own database in the same instance, which was done as a cheap hack.
2. Dev environment should not be visible externally, it currently is for simplicity of testing.
3. Proper setup for integration testing of Docker images and testing in general.
4. Access restrictions for git branches.
5. Longer commit digests for GCR labels.
6. Proper log management and monitoring.

### Tentative improvements

1. Storing code in Google Cloud which simplifies hooks, automation and error reporting.
2. Assigning `live` and `RCxxx` tags to git branches, to make that information available
outside GCR.
3. Helm or Ansible for templating deployment configurations.
