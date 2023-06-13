# Onboarding

Welcome to the waltti-apc developer team!

This document describes your first steps to get started.

Your first job is to improve this document and the other documentation that you use to get up to speed.

## Accounts

### Slack

Slack is our main communication channel.

Ask a development team member to add you to the following Slack Connect channels:

- `#waltti-silmä`: Communicate with LMJ and the municipalities.
- `#waltti-silmä-dev`: Communicate with developers while being able to ping LMJ or the municipalities if necessary.
- `#apc-pilot-partners`: Communicate with the pilot partners.

### GitHub

Ask a [waltti-apc team maintainer](https://github.com/orgs/tvv-lippu-ja-maksujarjestelma-oy/teams/waltti-apc/members) to add you as a member.
Tell them your GitHub account name.

### Google Cloud Platform

Ask LMJ personnel to add you into the [APC developer group](https://console.cloud.google.com/iam-admin/iam?folder=934719793254).
Tell them your work email address.

### StreamNative

Ask a team member to add you into the [waltti StreamNative organization](https://console.streamnative.cloud/users?org=waltti).
Use your work email address.

### CloudAMQP

Ask a team member to add you into the [Waltti CloudAMQP team](https://customer.cloudamqp.com/team) while they are logged into team Waltti.
Use your work email address.

### Preset

Ask a team member to add you into the [Waltti APC Preset team](https://manage.app.preset.io/app/teams/21db4948/members).
Use your work email address.

### Mergify

We use [Mergify](https://mergify.com/) to merge pull requests by Dependabot.
Mergify does not have its own bot account.
Instead, it uses a random organization member's credentials to e.g. rebase pull requests.

Log in with your GitHub account into Mergify at least once and authorize it to use your account, as well.

## Documentation

Documentation is primarily maintained in the [main repository](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc).
The documentation specific to running each microservice is maintained in the repositories of the corresponding microservices.
The main repository should have links to all the repositories that are used in the waltti-apc project.

Do not use GitHub Wiki as it does not provide any easier UX than simply adding text documents into the main repository.

## Project management

All issues created by a human are managed in the [GitHub Issues of the main repository](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc/issues).
Only bots might create issues in the repositories of the microservices.

The project is managed through a [Kanban board](https://github.com/orgs/tvv-lippu-ja-maksujarjestelma-oy/projects/1/views/1).

## Development

The microservices are written in TypeScript.

The code for the microservices has a lot of similarity.
The differences are in business logic.
Especially the main function, reading the configuration, handling Pulsar connections and supporting health checks looks similar in all of them.
We have not yet figured out which parts to move into a shared library and how to do it.

The README.md in each microservice follows a similar structure:

1. name,
1. description,
1. instructions for developing and running and
1. environment variables used for configuration.

The part of the CI/CD pipeline managed in the repository of each microservice has no microservice-specific parts.
It can be copied from one microservice to the next as such.

On success, the microservice repository CI/CD pipeline will push a Docker image into our [Docker Hub account](https://hub.docker.com/u/tvvlmj).
The credentials for Docker Hub are stored as GitHub organization secrets.
The corresponding Docker image repository will be automatically created on the first push.

### Creating new GitHub repositories

If you are creating a new repository from scratch:

1. If you believe the new repository will likely ever be used only within the waltti-apc project, use the prefix `waltti-apc-` in the repository name.
   Most of our repositories seem like this.
1. In many of our repositories, the purpose of the repository has been described in one sentence.
   That sentence is then used as:

   - the first paragraph of the repository `README.md`,
   - the GitHub repository description and
   - the `package.json` description.

1. Add the file LICENSE as the first commit.
   It should contain the `EUPL-1.2` license.
   You can copy the file from the [main repository](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc/blob/main/LICENSE).
1. Copy the repository settings from [waltti-apc-journey-matcher](https://github.com/tvv-lippu-ja-maksujarjestelma-oy/waltti-apc-journey-matcher/settings).
   If you use different settings, document those settings and the rationale.
1. Copy what you need from our existing microservices.

If you are forking an open-source repository from another project:

1. Check that it can be used as part of our `EUPL-1.2`-licensed project.
1. Use the license of the original repository if possible.
1. Do not change the name of the fork from the original unless it makes sense.

Link from this main repo to the new repository and vice versa.
