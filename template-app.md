# Template and Instructions for a new app

This lays out the generic layout of any new web app. Scale is optimized for small scale app with single developer. Some parts (e.g., lack of kubernetes, lack of ansible, instance types, etc.) will need to be re-evaluated if app has high usage volume or there are multiple developers involved.

## AWS

- separate deployments for prod and staging environments
- create a vpc per environment
  - region: us-west-2 (Oregon)
  - CIDR:
    - prod:  172.21.0.0/16
    - staging: 172.31.0.0/16
- subnets: one public for nginx (*.*.16.0/24), one private for app/db (*.*.18.0/24)
- internet gateway for frontend subnet
- backend network uses frontend nginx instance as NAT gateway (instance needs to be set up to do that)
- security groups:
  - frontend: allow inbound http (80), https (443), ssh (22); outbound everything
  - backend: allow inbound from nginx on fe network, outbound everything
- route tables:
  - frontend: internet gateway to allow all inbound and outbound
  - backend: route to EC2 instance on frontend as NAT gateway
- persistent IP address (Elastic IP) for frontend instance, that dns can point to
- persistent ebs volume for postgres data, attached to instance hosting postgres
- official postgres docker image has a feature to run an init SQL script if the database's data directory on filesystem is empty. Use that to run initialization sql script to create schema. Only runs the first time at bring-up - subsequent schema updates need to be manually done.
- use existing aws keypair (make sure public key is uploaded to aws in the correct region)
- ECR (Elastic Container Registry) to host docker images; Separate repos per environment.
- in the aws provider block, create default tags of "project" and "environment"
- t4g family instances tend to be cheaper, use them if possible (i.e., if using Amazon's ARM Graviton CPU is OK)
  - If intel CPUs are needed, consider t3 (Intel) or t3a (AMD, cheaper) family
  - t4g (or other graviton CPUs) require cross-compilation if the dev machine is Windows amd64 PC.

## Terraform

- Create a prod and a staging environment
- use modules to keep resource definitions in a common directory
- create separate directories for the environments with the main.tf and references to modules and tfvars in the per-environment directory. That way I can cd into that directory and do operations on the environment. Modules minimize the duplicated resource definitions
- In EC2 instances, use private_ip = cidrhost(...) to assign stable IP addresses to hosts that dont change from deploy to deploy. This makes it easy to reference hosts from other hosts
- create dns record to point to elastic ip within terraform if there is a terraform provider for dns registrar (e.g., cloudflare has one)

## App Layout

- nginx for reverse proxy in front of python or nodejs app servers
- nginx terminates tls
- db instance on postgresql in a container, running on the app server EC2 instance. We can scale it later to Aurora or RDS if the scale grows bigger
- an api server container, and all interactions between app server and db are through the APIs exposed by the api server. Think through api as part of design.
  - api may eventually be opened up to the internet as well, as a way of programmatic interactions
- Initially, during prototype, just use username on the login screen with no password as authentication. Eventually add auth0 as authn/authz.
- If using python, use flask + gunicorn, and use htmx and/or jinja2 for content
- If node js, use express
- create a "scripts" directory and put any automation in there (e.g., deploy script, image publish script, certbot initial bootstrap dance for certs, etc.)

## docker and docker-compose

- use docker container for every role, and docker compose to bring up the set of containers
- use profiles to separate containers that go into different instances. Deploy the right profiles on the right machines in prod, and deploy all profiles on the dev machine onebox
- Docker files are layered, with base docker compose and per-environment overlay

## TLS and Certs

- use Let's Encrypt and Certbot, with nginx directory in website's path used for cert validation (not DNS based validation)
- create a script to do the bootstrap with nginx and certbot dance on port 80 and setting up the cron job for renewal. The script should download the certs in a tarball to local computer, so if the environment is destroyed and recreated, we dont have to fetch new certs again from let's encrypt. This matters for stage, because we will do terraform destroy against stage, which will wipe the fetched certs on the ec2 instance.

## Cloudflare

- consider cloudflare for static resources and static websites
- cloudflare also has good DNS api's and programmability, and has a terraform provider

## Secrets

- environment variable files. These contain non-sensitive configuration information.
  - an example file that explains all values
  - files for local and prod and staging environments with actual values. these are gitignored
- secrets directory that contains one file per secret, consumed by docker. gitignored.

## Verification

- claude should verify changes after edits. Explicitly lay out the verifications that make sense, in a skill called validate-all

## Project Structure and Claude Code


- create a docs/ directory, and put design docs and claude plan output in there.
- Create an architecture.md in docs/ directory that lays out the technical architecture and design and component details of the project. Reference it in claude.md
- create a scripts/ directory that has ad-hoc automation scripts (e.g., deploy, build, publish images, etc., etc., as needed)
- create a LICENSE file that has an all rights reserved/not-open-source copyright notice
- set up an AGENTS.md. The purpose of it is not to document the architecture, but to document instructions to claude on how to operate. Put a reference to the architecture.md in agents.md
- Set up an CLAUDE.md that contains nothing and points to AGENTS.md
- create a skills directory under .claude
- create a gitignore
- create a .gitattributes, with EOL set to LF for *.sh files and all Dockerfile files

