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


<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 19 03 PM" src="https://github.com/user-attachments/assets/a8536cef-37a0-4a41-84a3-49654277c728" />


### Redis Server

* OS: Ubuntu 24.04
* Redis: 7.0.15
* Private IP: `172.31.11.179`
* Service: `redis-server`

---

<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 20 32 PM" src="https://github.com/user-attachments/assets/c6629b27-0061-45ee-928c-09cbcc895787" />


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


<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 23 48 PM" src="https://github.com/user-attachments/assets/7bc1df02-35f2-460b-97cc-3c65dbc07e68" />

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


<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 24 38 PM" src="https://github.com/user-attachments/assets/b829f587-185e-41de-affe-4371f849e1de" />

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


<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 25 50 PM" src="https://github.com/user-attachments/assets/b106c275-cf60-4b54-ad71-b1e1560e841d" />

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

<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 26 48 PM" src="https://github.com/user-attachments/assets/a19128df-2500-47f8-9050-4019ba965afd" />


## Shared Library Stages

### 1. Clone

Clones the Redis Ansible project from GitHub.

### 2. User Approval

Jenkins asks for approval before production deployment.

```text
Approve Redis deployment to prod?
```


<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 31 19 PM" src="https://github.com/user-attachments/assets/5d4b1ad5-a5e4-4b9c-a843-c22c5283ab84" />

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

<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 32 57 PM" src="https://github.com/user-attachments/assets/82d50b66-a08b-4e78-9372-f953187e9002" />


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
<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 32 57 PM" src="https://github.com/user-attachments/assets/3bcf5633-db5b-4a95-a55c-1e661a6f46cf" />


### Slack Webhook

Credential ID:

```text
slack-webhook
```

---
<img width="1440" height="900" alt="Screenshot 2026-09-28 at 11 35 17 PM" src="https://github.com/user-attachments/assets/a8ec4793-16de-4d0a-997d-61355cd546fd" />


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

