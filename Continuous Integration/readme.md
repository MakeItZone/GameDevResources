**Table of Contents**
- [Jenkins](#jenkins)
  - [Master Installation](#master-installation)
  - [Agents](#agents)
  - [Old School Agents](#old-school-agents)
  - [Jenkins for Unity](#jenkins-for-unity)
  - [Jenkins Scripts/Pipelines](#jenkins-scriptspipelines)

# Jenkins
Jenkins is a mature open source CI server.
It can be installed stand-alone, but the most common way now seems to be via Docker.

## Master Installation
- https://www.jenkins.io/doc/book/installing/docker/
- https://www.cloudbees.com/blog/how-to-install-and-run-jenkins-with-docker-compose
- [docker set up, including master and (dynamic?) agents](https://www.bogotobogo.com/DevOps/Docker/Docker-Jenkins-Master-Slave-Agent-ssh.php)

## Agents
- [Docker containers as agents](https://devopscube.com/docker-containers-as-build-slaves-jenkins/)
- Question: how to have Jenkin's master set up docker containers on multiple other hosts? Setting in docker plugin? Install multiple instances of the Docker plugin?

## Old School Agents
- looks like Jenkins can use docker/docker swarm directly, so can dynamically spin up agents (also on other hosts) as needed
- not sure if that works for Windows agents (with full gfx access)
  - here are the docs for [setting up explicit, full time agents](https://www.jenkins.io/doc/book/using/using-agents/)
  - 

## Jenkins for Unity
- [GameCI- Docker images for Unity](https://game.ci) - most up-to-date and maintained; active dischord which is very helpful
  - they suggest looking at the [GitLab integration instructions](https://game.ci/docs/gitlab/getting-started), which should be mod'able to work with standalone Jenkins
- [Series of posts on how to set it up](https://andrewfray.wordpress.com/tag/jenkins/)
- [Another article re using Jenkins for Unity](https://smashriot.com/unity-build-automation-with-jenkins/)

## Jenkins Scripts/Pipelines
Original Jenkins had the CI process for a project be defined by the GUI on the master, and lived outside of the project code (and outside of source control- from memory.)

Jenkin's files make the build configuration "as code", that can live with the source code- much better way of doing things!

- [docker pipeline info](https://docs.cloudbees.com/docs/admin-resources/latest/plugins/docker-workflow)
- 