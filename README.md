# Jenkins-Pipelines

This project gathers all the [Jenkins](https://www.jenkins.io/) pipeline definitions (`Jenkinsfile`)
used by the Continuous Integration and Continuous Delivery (CI/CD) platform of
[Silverpeas](https://www.silverpeas.org).

Each pipeline describes the different stages of a given Jenkins job: constructing and publishing a
build version of a project, releasing a stable version, generating and publishing the documentation
and the community web site, producing the Docker images, or testing the installation and the upgrade
of the platform.

## Goal

The Silverpeas platform is made up of several independent Git projects (Silverpeas-Core,
Silverpeas-Components, Silverpeas-Looks, Silverpeas-Assembly, Silverpeas-Setup,
Silverpeas-Distribution, ...) plus a set of satellite libraries and tools. Building, releasing and
delivering all of them consistently requires an automated and reproducible chain of jobs.

The purpose of this project is to define, to version and to share, in a single place, the whole
description of that chain so that:

* the CI/CD workflow of Silverpeas is documented, reviewable and traceable through the SCM history;
* any job of the Silverpeas Jenkins server just refers to a `Jenkinsfile` of this project instead of
  embedding its own script;
* the builds are reproducible: almost all the pipelines run within a dedicated Docker image
  (`silverpeas/silverbuild`) in order to containerize the build from the host OS and to pin the
  versions of the JDK, of Maven and of the Wildfly instance used by the integration tests.

## Organization

The pipelines are located in the `src` directory, and are split by purpose:

```
src/
├── builds/     pipelines constructing a build (non-stable) version of a project
├── releases/   pipelines releasing a stable version of a project or an artifact
├── tests/      pipelines validating the installation and the upgrade processes
├── pipeline.gdsl
└── pipeline-syntax.gdsl
```

### `src/builds`

Pipelines producing a *build version*, id est a non-stable version timestamped with the date of the
build (`<next release>-build<yyMMdd>`), deployed into the `builds` repository of the Silverpeas Nexus
server. A build version for which the tests and the quality analysis weren't skipped is a candidate
for a release of a stable version.

| Pipeline             | Purpose                                                                                    |
|----------------------|--------------------------------------------------------------------------------------------|
| `project`            | the Silverpeas Project POM (parent POM of all Silverpeas projects) and its two BOM projects |
| `silverpeas`         | Silverpeas itself: Core, Components, Assembly, Setup, Distribution and Looks                |
| `mobile`             | Silverpeas Mobile                                                                           |
| `sso`                | the SSO library for Silverpeas                                                              |
| `wbe`                | the Web Browser Edition library for Silverpeas                                              |
| `jackrabbit-jca`     | the RAR of JackRabbit JCA customized for Silverpeas                                         |
| `jcr-access-control` | the JCR access control library for JackRabbit                                               |
| `project-doc`        | the web site of the open-source community of Silverpeas                                     |
| `coverity-scan`      | the Coverity static analysis of the Silverpeas source code                                  |

### `src/releases`

Pipelines releasing a stable version, usually from a given build version, and publishing it into the
`releases` repository of the Silverpeas Nexus server. Beside the Maven artifacts, they also take
care of the delivery of the platform:

| Pipeline                               | Purpose                                                                        |
|----------------------------------------|--------------------------------------------------------------------------------|
| `project`, `kernel`                    | the project definition (parent POM and BOMs) and the Silverpeas Kernel library  |
| `silverpeas`                           | a stable version of Silverpeas; it triggers most of the pipelines below         |
| `sso`, `wbe`                           | the SSO and the Web Browser Edition libraries                                   |
| `jackrabbit-jca`, `jcr-access-control` | the JackRabbit related artifacts                                                |
| `silverpeas-doc`                       | the documentation of a new version of Silverpeas and the community web site     |
| `izpack`                               | the IzPack installer of a stable version of Silverpeas                          |
| `docker-build`                         | the Docker image used to build the Silverpeas projects                          |
| `docker-dev`                           | the Docker image providing a reproducible development environment               |
| `docker-test`                          | the Docker image of a version of Silverpeas for testing purpose                 |
| `docker-prod`                          | the production-ready Docker image, submitted to the Docker official images      |

### `src/tests`

Pipelines checking, against the last build version of Silverpeas, that the installation process and
the upgrade process of the platform work as expected.

### The GDSL files

`pipeline.gdsl` and `pipeline-syntax.gdsl` describe the Jenkins pipeline DSL (steps, global
variables, environment variables) to the Groovy support of IntelliJ IDEA. They are here only to
provide code completion and to avoid false errors when editing a `Jenkinsfile` in the IDE; they are
never executed by Jenkins.

## Conventions expected by the pipelines

Most of the pipelines rely upon conventions on the Jenkins side; they are documented in the header
comment of each `Jenkinsfile`. The main ones are:

* **Job naming**: a job running a build pipeline is expected to be named
  `[ANY WORD]_[TYPE_BRANCH]_[ANY WORD]`, with `TYPE_BRANCH` being `Master` for the main development
  branch, `Stable` for the branch of the current stable version, or the name of an older stable
  branch. The pipeline figures out from it both the SCM branch to check out and the version of the
  Docker image to use.
* **Environment variables**: `STABLE_BRANCH` (the SCM branch in which the current stable version is
  maintained) and `IMAGE_FOR_STABLE` (the version of the Docker image to use for that stable
  version) have to be set at the Jenkins level.
* **Job dependencies**: the pipelines exchange their result through a YAML build report
  (`build.yaml`) archived as an artifact of the job. For instance, the build of Silverpeas fetches
  the report of the `Silverpeas_Project_Definition_AutoDeploy` job to know the version of the parent
  POM to use, and the release pipelines fetch the report of the corresponding `*_AutoDeploy` job to
  know the build version from which the release has to be done.

## Usage

In Jenkins, declare a *Pipeline* job whose definition is *Pipeline script from SCM*, referring to
this Git repository and to the path of the wished `Jenkinsfile` (for example
`src/builds/silverpeas/Jenkinsfile`). Then set the environment variables and the parameters expected
by that pipeline, as stated in its header comment.

## License

This project is released under the terms of the GNU General Public License version 3; see the
[LICENSE](LICENSE) file.
