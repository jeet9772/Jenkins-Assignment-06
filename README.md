
# Jenkins Ansible Shared Library

## Overview

This project demonstrates a reusable **Jenkins Shared Library** for automating Ansible deployments.

The pipeline is designed to perform the complete deployment workflow:

**Clone → Configuration → User Approval → Ansible Playbook Execution → Slack Notification**

The pipeline receives its required inputs from a centralized `config.properties` file, making the shared library reusable across different environments and projects.

---

## Architecture

```text
                    GitHub
                      |
          +-----------+-----------+
          |                       |
          v                       v
 ansible-shared-library     ansible-demo-project
          |                       |
          |                       |
          +---------- Jenkins ----+
                       |
                       v
                Load Configuration
                       |
                       v
                  Clone Code
                       |
                       v
                 User Approval
                       |
                       v
             Ansible Playbook
                       |
                       v
                  AWS EC2
                       |
                       v
               Slack Notification
```

---

## Repository Structure

### Shared Library Repository

```text
ansible-shared-library/
│
└── vars/
    └── ansibleDeploy.groovy
```

<img width="1440" height="900" alt="Screenshot 2026-09-23 at 1 12 30 PM" src="https://github.com/user-attachments/assets/0d255436-7a6a-4474-bcb4-35bbf9428ad9" />


### Ansible Demo Project

```text
ansible-demo-project/
│
├── Jenkinsfile
├── config.properties
│
└── env/
    └── prod/
        ├── playbook.yml
        └── inventory
```

---

## Technologies Used

* Jenkins
* Jenkins Shared Libraries
* Ansible
* Git & GitHub
* AWS EC2
* SSH
* Slack
* Groovy
* YAML

---

## Configuration

All required pipeline inputs are maintained in `config.properties`.

```properties
SLACK_CHANNEL_NAME=build-status
ENVIRONMENT=prod
CODE_BASE_PATH=env/prod
ACTION_MESSAGE=Ansible deployment completed successfully
KEEP_APPROVAL_STAGE=true
```

### Configuration Parameters

| Parameter             | Description                                    |
| --------------------- | ---------------------------------------------- |
| `SLACK_CHANNEL_NAME`  | Slack channel where the notification is sent   |
| `ENVIRONMENT`         | Deployment environment                         |
| `CODE_BASE_PATH`      | Location of the Ansible playbook and inventory |
| `ACTION_MESSAGE`      | Slack notification message                     |
| `KEEP_APPROVAL_STAGE` | Controls whether manual approval is required   |

---

## Jenkinsfile

The project Jenkinsfile loads the shared library and calls the reusable deployment function.

```groovy
@Library('ansible-shared-library') _

ansibleDeploy('config.properties')
```

This keeps the project Jenkinsfile simple while the actual deployment logic remains inside the shared library.

---

## Shared Library Workflow

### 1. Load Configuration

The pipeline reads `config.properties` using the Jenkins Pipeline Utility Steps plugin.

```groovy
config = readProperties file: configFile
```

The configuration is then used throughout the pipeline.

---

### 2. Clone

The source code is checked out from the configured SCM repository.

```groovy
checkout scm
```

---

### 3. User Approval

Before deployment, Jenkins asks for manual approval when:

```text
KEEP_APPROVAL_STAGE=true
```

The approval message identifies the target environment.

```text
Approve deployment to prod?
```

This provides a manual control before executing the Ansible deployment.

---

### 4. Ansible Playbook Execution

The pipeline executes the Ansible playbook using the configured path and inventory.

```bash
ansible-playbook env/prod/playbook.yml \
-i env/prod/inventory \
-e environment=prod \
--ssh-common-args='-o StrictHostKeyChecking=no'
```

The Ansible inventory contains the target EC2 instance and SSH connection details.

---

## Ansible Playbook

The demonstration playbook performs OS-specific tasks:

```yaml
---
- name: Load OS specific variables
  include_vars: "{{ ansible_os_family }}.yml"

- name: Run tasks for CentOS/RedHat
  include_tasks: redhat.yml
  when: ansible_os_family == "RedHat"

- name: Run tasks for Ubuntu/Debian
  include_tasks: ubuntu.yml
  when: ansible_os_family == "Debian"
```

### Deployment Result

After successful execution, the following file is created on the target EC2 instance:

```text
/tmp/ansible-demo.txt
```

This confirms that Jenkins successfully triggered Ansible and Ansible successfully connected to and executed tasks on the EC2 instance.

---

## Slack Notification

After successful playbook execution, the shared library sends a notification to the configured Slack channel.

```groovy
slackSend(
    channel: config.SLACK_CHANNEL_NAME,
    message: config.ACTION_MESSAGE
)
```

For the current configuration, the notification is sent to:

```text
#build-status
```

with the message:

```text
Ansible deployment completed successfully
```

---

## Jenkins Pipeline Flow

```text
START
  |
  v
Load Configuration
  |
  v
Clone Repository
  |
  v
User Approval
  |
  v
Execute Ansible Playbook
  |
  v
Deploy to AWS EC2
  |
  v
Send Slack Notification
  |
  v
SUCCESS
```

---

## Prerequisites

Before running the pipeline, ensure the following are available:

* Jenkins
* Ansible installed on the Jenkins execution node
* Git configured in Jenkins
* Jenkins Pipeline Utility Steps plugin
* Slack Notification plugin
* AWS EC2 instance
* SSH access from Jenkins to the EC2 instance
* Required Jenkins Shared Library configuration
* Required Slack credentials/configuration

---

## Jenkins Shared Library Configuration

The shared library is configured in:

```text
Manage Jenkins
→ System
→ Global Trusted Pipeline Libraries
```

### Configuration

```text
Name: ansible-shared-library
Default Version: main
SCM: Git
Repository:
https://github.com/bhumi262/jenkins-shared-library.git
```

---

## Benefits of Using a Shared Library

Using a Jenkins Shared Library provides:

* Reusable pipeline logic
* Centralized CI/CD automation
* Reduced Jenkinsfile complexity
* Consistent deployment workflow
* Configuration-driven execution
* Easier maintenance
* Reusability across multiple projects and environments

---

## Conclusion

This project demonstrates how Jenkins Shared Libraries can be used to create a reusable and configuration-driven Ansible deployment pipeline.

The implementation automates the complete workflow from source code checkout to manual approval, Ansible execution on AWS EC2, and Slack notification, providing a simple foundation for scalable CI/CD automation.
