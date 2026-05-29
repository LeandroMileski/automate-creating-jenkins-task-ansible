# Automated Jenkins Provisioning with Ansible

One command spins up a fully configured, build-ready Jenkins server on AWS — network, security, and tooling included.

## The business problem

Teams that need Jenkins on demand often lose hours to manual setup: launching a server, wiring up networking, installing Java, Jenkins, Docker, and Node.js, and chasing the configuration drift that comes with doing it by hand. Every manual instance is a chance for inconsistency, a security gap, or a forgotten step.

## The value delivered

This project automates the entire lifecycle with a single Ansible run, turning a multi-hour manual chore into a repeatable, ~5‑minute operation:

- **Faster delivery** — a new, identical Jenkins environment is available in minutes, on demand, whenever a team needs one.
- **Consistency & reliability** — every instance is provisioned identically from code, eliminating "works on my server" drift and manual error.
- **Lower cost & overhead** — no manual engineer time per setup; instances are disposable and reproducible, so teams create and tear down freely.
- **Security by default** — networking and firewall rules are defined in code and version-controlled, so access is intentional and auditable rather than ad hoc.
- **Build-ready out of the box** — Docker and Node.js/npm are pre-installed and integrated, so CI/CD pipelines can build containers and JavaScript apps immediately.

## What it provisions

A single playbook handles both infrastructure and configuration end to end:

- **Infrastructure (AWS):** dedicated VPC, public subnet, internet gateway, route table, security group, and an EC2 instance — built from scratch for full control.
- **Configuration (on the server):** Java 21, Jenkins (running and enabled), Docker (with the Jenkins user authorized to run containers), and Node.js + npm.
- **Verification:** automated checks confirm every service is live and the Jenkins UI is responding before the run completes.

## How it works

The automation runs in two stages within one playbook:

1. **Provision** — runs locally against the AWS API to create the network and launch the server, then registers the new instance dynamically so it can be configured in the same run.
2. **Configure** — connects to the fresh server over SSH and installs and starts the full Jenkins stack.

## Tech stack

Ansible · AWS (EC2, VPC) · Jenkins · Docker · Node.js · Amazon Linux 2023

## Run it

```bash
ansible-galaxy collection install amazon.aws
ansible-playbook create-jenkins.yml
```

Jenkins is then reachable at `http://<server-public-ip>:8080`.
