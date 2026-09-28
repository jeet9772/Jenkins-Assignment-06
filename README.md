# Jenkins Assignment 6 – Ansible Shared Library

## Objective

Create a Jenkins Ansible Shared Library for Redis deployment with:

1. Clone
2. User Approval
3. Playbook Execution
4. Notification

---

## Architecture

```text
GitHub
  |
  v
Jenkins Pipeline
  |
  v
Ansible Shared Library
  |
  +--> Clone
  |
  +--> User Approval
  |
  +--> Ansible Playbook
  |
  +--> Slack Notification
  |
  v
Redis EC2 Server
```

---

## Infrastructure

### Jenkins Server

* OS: Ubuntu 24.04
* Jenkins: 2.568.3
* Ansible: 2.16.3
* Java: 21

### Redis Server

* OS: Ubuntu 24.04
* Redis: 7.0.15
* Private IP: `172.31.11.179`
* Service: `redis-server`

---

## GitHub Repositories

### Ansible Project

```text
https://github.com/jeet9772/Redis_Ansible_Assignment6.git
```

Structure:

```text
Redis_Ansible_Assignment6/
├── inventory
├── playbook.yml
├── ansible.cfg
└── config/
    └── redis-prod.conf
```

### Shared Library

```text
https://github.com/jeet9772/Jenkins-Ansible-Shared-Library.git
```

Structure:

```text
Jenkins-Ansible-Shared-Library/
└── vars/
    └── ansibleDeploy.groovy
```

---

## Configuration

File:

```text
config/redis-prod.conf
```

```text
SLACK_CHANNEL_NAME=build-status
ENVIRONMENT=prod
CODE_BASE_PATH=env/prod
ACTION_MESSAGE=Redis deployment completed successfully
KEEP_APPROVAL_STAGE=true
```

---

## Jenkins Pipeline

```groovy
@Library('ansible-shared-library') _

pipeline {
    agent any

    stages {
        stage('Redis Deployment') {
            steps {
                script {
                    ansibleDeploy(
                        gitUrl: 'https://github.com/jeet9772/Redis_Ansible_Assignment6.git',
                        gitBranch: 'main',
                        configFile: 'config/redis-prod.conf',
                        inventory: 'inventory',
                        playbook: 'playbook.yml'
                    )
                }
            }
        }
    }
}
```

---

## Shared Library Stages

### 1. Clone

Clones the Redis Ansible project from GitHub.

### 2. User Approval

Jenkins asks for approval before production deployment.

```text
Approve Redis deployment to prod?
```

### 3. Playbook Execution

Ansible executes the Redis playbook on the Redis EC2 server.

Result:

```text
Redis service status: active
```

### 4. Notification

After deployment, Jenkins sends the deployment message to the Slack:

```text
#build-status
```

---

## Ansible Playbook

The playbook:

* Installs Redis
* Enables Redis service
* Starts Redis service
* Checks Redis status
* Displays Redis status

Successful result:

```text
ok=5
changed=0
unreachable=0
failed=0
```

---

## Jenkins Credentials

### SSH Key

```text
/var/lib/jenkins/Jeet11.pem
```

### Slack Webhook

Credential ID:

```text
slack-webhook
```

---

## Final Result

The Jenkins pipeline successfully performs:

```text
Clone
  ↓
User Approval
  ↓
Ansible Playbook Execution
  ↓
Redis Deployment
  ↓
Slack Notification
```

Redis deployment was successfully verified with:

```text
Redis service status: active
```

## Conclusion

Jenkins Ansible Shared Library was created and integrated with the Redis deployment pipeline. The pipeline supports configuration-based deployment, manual approval, Ansible execution, and Slack notification.

