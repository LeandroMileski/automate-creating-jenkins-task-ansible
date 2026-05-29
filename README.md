<img width="1668" height="563" alt="image" src="https://github.com/user-attachments/assets/9c20f45a-9a28-4439-aaaf-611600080823" />
<img width="1676" height="804" alt="image" src="https://github.com/user-attachments/assets/0aa8cbc8-608f-499b-97da-221db23246b0" />

<img width="1666" height="377" alt="image" src="https://github.com/user-attachments/assets/99ead1a6-3ed0-4ba3-b25e-38498a4ea377" />

<img width="1919" height="951" alt="image" src="https://github.com/user-attachments/assets/240c92a9-accf-4c6f-a03b-d81f94568b00" />

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
- **Deployment flexibility** — the same outcome on Amazon Linux, Ubuntu, or as a portable Docker container, so the approach fits whatever a team already runs.

## Playbooks

Three interchangeable playbooks deliver the same build-ready Jenkins, adapted to different environments:

| Playbook | What it does | Best for |
|---|---|---|
| `create-jenkins.yml` | Provisions AWS infrastructure and installs Jenkins natively on **Amazon Linux 2023** (Java 21, Docker, Node.js + npm). | AWS-native, RHEL-family shops. |
| `create-jenkins-ubuntu.yml` | Same outcome on an **Ubuntu 24.04** server, using apt and the Debian Jenkins repository. | Ubuntu/Debian environments. |
| `create-jenkins-docker.yml` | Provisions the server, installs only Docker, and runs Jenkins as a **container** (`jenkins/jenkins:lts`) with persistent storage and access to the host Docker daemon. | Portable, disposable, container-first setups. |

Each playbook is self-contained: it builds the full network (VPC, public subnet, internet gateway, route table, security group), launches the EC2 instance, registers it dynamically, configures it over SSH, and verifies that every service is live before finishing.

### About the Docker variant

The container version mirrors this Docker command, expressed declaratively in Ansible so it is idempotent and repeatable:

```bash
docker run --name jenkins -p 8080:8080 -p 50000:50000 -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/bin/docker:/usr/bin/docker \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

Mounting the host's Docker socket and binary lets Jenkins run Docker commands inside its own pipelines; the `jenkins_home` named volume keeps Jenkins data across container restarts.

## Prerequisites

On the control machine (where you run Ansible):

- **Ansible** and **Python 3** installed.
- **boto3** and **botocore** (the AWS SDK the modules use):
  ```bash
  pip install boto3 botocore
  ```
- **Required Ansible collections:**
  ```bash
  ansible-galaxy collection install amazon.aws
  ansible-galaxy collection install community.docker   # only for the Docker variant
  ```
- **AWS credentials** with permission to manage EC2/VPC, exported in the environment (or configured via `aws configure`):
  ```bash
  export AWS_ACCESS_KEY_ID="..."
  export AWS_SECRET_ACCESS_KEY="..."
  export AWS_REGION="eu-west-1"
  ```
- **An SSH key pair** at `~/.ssh/id_rsa` / `~/.ssh/id_rsa.pub`. The playbooks import the public key into AWS automatically; the private key is used to connect to the new server.

## Run it

```bash
# Native on Amazon Linux 2023
ansible-playbook create-jenkins.yml

# Native on Ubuntu 24.04
ansible-playbook create-jenkins-ubuntu.yml

# Jenkins as a Docker container
ansible-playbook create-jenkins-docker.yml
```

Each run prints the Jenkins URL and the initial admin password when it completes. Jenkins is reachable at `http://<server-public-ip>:8080`.

## Tech stack

Ansible · AWS (EC2, VPC) · Jenkins · Docker · Node.js · Amazon Linux 2023 · Ubuntu 24.04
