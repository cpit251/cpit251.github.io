---
title: "Continuous Integration (CI)"
date: 2023-01-05T22:34:22+03:00
lastmod: 2023-01-06T09:30:51+03:00
weight: 3
draft: false
description: "This lecture note will guide you through everything you need to know about continuous integration (CI). We’ll go through an example of how to use a CI service (circleCI) with a version control service (GitHub) to automate the process of building and testing your software."
---

![Continuous Integration (CI)](/images/notes/ci/continuous-integration-CI.png)

## What is continuous integration?

Continuous integration (CI) is the practice of automating the integration of changes from multiple contributors into a central repository. When new code gets checked in by a developer, it will go through the CI pipeline before it gets integrated into the central repository. The CI pipeline may include running automated tests, test coverage, code style checkers, quality tests, review tools, and more. The CI will make sure that the pipeline is successful prior to merging the new changes to the central repository. If the pipeline has failed at any stage, the changes are rejected and will be reported to the team, so these changes do not make it to the central repository or main branch in the version control system (e.g., git). With CI, every developer in the team commits daily and pushes to the central repository, each commit triggers the CI pipeline to run automated tests and builds. When the pipeline is unsuccessful, errors are reported immediately, so developers can have a high level of observability and confidence in resolving these errors.

CI is an effective agile strategy and a powerful tool that enables teams to develop software rapidly with faster releases without sacrificing quality. Teams use CI to ensure that software is built successfully with safeguards from new regressions or unexpected failures.


## Examples of CI services and servers?
There are many Continuous Integration (CI) services out there that run as a Software-as-a-Service (SaaS). Examples of such CI services include [Travis CI](https://www.travis-ci.com), [CircleCI](https://circleci.com/), and [GitHub Actions](https://docs.github.com/en/actions).
 You can also host your own CI server on a virtual private server of your own when using an open source automation server such as [Jenkins](https://www.jenkins.io/), [Agola](https://agola.io/), and [GoCD](https://www.gocd.org/).


## Create a CI pipeline using CircleCI
CircleCI is a modern and powerful CI tool that is easy to setup and configure. It features a free plan that's good for learning and small projects.

#### Requirements
- GitHub account.
- CircleCI account.
- A repository hosted on GitHub. I'm using a simple [maven project for a Java application]()

### 1. Sign up for a free account on CircleCI with github

Go to [circleci.com](https://circleci.com) and sign up for an account with your GitHub account.
![Continuous Integration (CI) CircleCI create an account](/images/notes/ci/step-0.png)
![Continuous Integration (CI) CircleCI create an account](/images/notes/ci/step-1.png)

Authorize Circleci to access your repositories on GitHub.

![Continuous Integration (CI) CircleCI create an account](/images/notes/ci/step-3.png)
![Continuous Integration (CI) CircleCI create an account](/images/notes/ci/step-4.png)

### 2. Choose or create a new project
Next, we’ll need an app to run our CI pipeline (build, test, and deploy). You can use any project you like. I’ve created a [simple Java app with Maven](https://github.com/cpit251/unit-testing-coverage-demo) and added Junit tests. You can easily create an new Maven project in IntelliJ IDEA by doing the following:
  - Go to **File** -> **New**  -> **Project**
  - Create a Java project with maven using the archetype generator `maven-archetype-quickstart`, which generates a sample Maven project:
  ![create a Java project with Maven](/images/notes/ci/idea-new-maven.png)
  - Next, add unit tests as discussed in [a previous lecture note](/notes/unit-testing).

### 3. Integrate your project with CircleCI
From the CircleCI dashboard, go to the **Projects** section, choose your GitHub repository, and click **"Set Up Project"**.

![Continuous Integration (CI) CircleCI setting up a new project](/images/notes/ci/step-5.png)

### 4. Configure the CI pipeline
CircleCI requires a configuration file to be stored at a hidden directory under the root project directory. This configuration file is written in YAML and stored at `.circelci/config.yml`. You will be asked to create one, use an existing one, or start from a template. Select the last option, to start from a `config.yml` template as shown below.

![Continuous Integration (CI) CircleCI setting up a new project](/images/notes/ci/step-6.png)

You will see a list of templates to choose from, select **Java (Maven)** since our java app is built with maven.

![Continuous Integration (CI) CircleCI setting up a new project](/images/notes/ci/step-7.png)

In the template file, find the line that starts with docker. We need to change the docker image that will be used to run our CI pipeline to match the version that our project is targeting as defined in the `pom.xml` file. I will change it to `18.0` to match our `pom.xml` in the sample project. Click download to download the template config file.

![Continuous Integration (CI) CircleCI setting up a new project](/images/notes/ci/step-8.png)


Next, create a directory/folder named `.circleci` under your project root directory (next to the `pm.xml` file) and place the `config.yml` file inside it.

![Continuous Integration (CI) CircleCI setting up a new project](/images/notes/ci/step-9.png)


Now we are ready to add this config file to our git repository. Commit the changes and push them to GitHub.

```bash
git add .circleci/config.yml
git commit -m 'add circleci config'
git push origin main
```
- ![Continuous Integration (CI) CircleCI setting up a new project](/images/notes/ci/step-10.png)


CircleCI will create a _build-and-test_ pipeline that is triggered any time we commit new changes to our repo on GitHub. Wait for the pipeline to kick in and succeed as shown below.

![Continuous Integration (CI) CircleCI running the first pipeline](/images/notes/ci/step-11.png)


Congratulations 🎉, your pipeline is successful and green. If you do not see a green pipeline, then check the errors in either the config file or your project (depending on what error message you get). If the pipeline is green, then we can keep working on pur project and continue to make changes with confidence since the CI pipeline is passing. 

Note that sometimes we like to make changes in a different branch other than the main branch, so if that was the case then we can merge that branch into the main branch once the pipeline is passing.

## Adding Status Badges

Finally, we wanted to add a badge that shows the build status (passed or failed) in the README file, so team members can quickly see build statuses in the repo's landing page on GitHub.

To create a badge, you need the following information:
To generate your own status badge, you will need to know, and substitute, the following variables in the code snippets below:

- <PROJECT_NAME> The name of the repo on GitHub. Example: `unit-testing-coverage-demo`
- <ORG_NAME> The organization or user name the project's repo belongs to. Example `cpit251` is the owner of our sample project on GitHub.
- <VCS> The version control system provider (gh for “github”)
- <LINK> The link you want the status badge to go to when clicked (example: the pipeline overview page)
- <circle-token> [Optional] An API token is required for private repos.

Before we create a badge, we wanted to add a `README.md` file to our project. create a new file named `README.md`.

![Continuous Integration (CI) CircleCI add a README file and a status badge](/images/notes/ci/step-12.png)


Next, add the following markdown code to the `README.md` file after replacing the values between `<>` with the values that match your repo:

```markdown
[![<ORG_NAME>](https://circleci.com/<VCS>/<ORG_NAME>/<PROJECT_NAME>.svg?style=svg)](<LINK>)
```

If your repo is private, then add the following code after replacing the values between `<>` with the values that match your repo:

```
[![CircleCI](https://circleci.com/<VCS>/<ORG_NAME>/<PROJECT_NAME>.svg?style=svg&circle-token=<YOUR_STATUS_API_TOKEN>)](<LINK>)
```


For the public sample project used here, I have added the following markdown code:

```
## Unit Testing and Coverage Example

[![cpit251](https://circleci.com/gh/cpit251/unit-testing-coverage-demo.svg?style=svg)](https://app.circleci.com/pipelines/github/cpit251/unit-testing-coverage-demo)
```

This should be rendered as shown below:

[![cpit251](https://circleci.com/gh/cpit251/unit-testing-coverage-demo.svg?style=svg)](https://app.circleci.com/pipelines/github/cpit251/unit-testing-coverage-demo)

## Conclusion

You've learned how to create a CI pipeline using a modern CI service, CircleCI, with a version control service, GitHub, to automate the process of building and testing your project. You're expected to have unit testing and a CI pipeline in your project and all future assignments.

