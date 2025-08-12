# Jenkins Pipeline: Ansible Deployment to EC2
This Jenkins pipeline automates the process of deploying an Ansible playbook to a remote EC2 instance using an SSH private key stored securely in Jenkins credentials.

## 📌 Overview
The pipeline performs the following steps:

1. Checks out code from the specified Git branch.

2. Retrieves an SSH private key from Jenkins credentials and configures it for use.

3. Adds the EC2 host to known_hosts to avoid interactive SSH prompts.

4. Generates an Ansible inventory file with the target host IP.

5. Runs the Ansible playbook on the target EC2 instance.


## 🛠 Prerequisites
Before running this pipeline, ensure the following are set up:

## Jenkins Plugins

- Git Plugin

- Credentials Binding Plugin

- Ansible Plugin (optional if running Ansible via shell)

## Jenkins Credentials

- Store your SSH private key as a Secret File:

  1. Go to Jenkins → Manage Jenkins → Credentials → Global Credentials (or folder-specific).

  2. Add a new Secret File with:

  - ID: Ansible_Private_Key (or update in the pipeline if you use a different ID).

  - The file should be your id_rsa private key used to connect to the EC2 instance.

- (Optional) Store the target EC2 IP as a Secret Text credential in Jenkins, then:

```groovy
TARGET_HOST_IP = credentials('TARGET_HOST_IP')
```

## Installed Tools on Jenkins
  - git
  - ssh
  - ansible

## ☁ EC2 Instance Configuration
- Add the public key (matching the stored private key) to:

```bash
~/.ssh/authorized_keys
```
of the ubuntu user (or your preferred SSH user).

- Allow inbound SSH traffic in the EC2 security group from your Jenkins server’s IP.

## 🔄 Pipeline Stages
## 1. Checkout Code
```groovy
git branch: 'main', url: 'https://your-repo-url.git'
```
Pulls the repository containing your Ansible playbook.

## 2. Setup SSH Key
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cp "$EC2_KEY_FILE" ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa
```
Retrieves the SSH key from Jenkins credentials and sets secure permissions.

## 3. Setup SSH Host
```bash
ssh-keyscan -H "$TARGET_HOST_IP" >> ~/.ssh/known_hosts
```
Adds the EC2 host’s public key fingerprint to known_hosts to avoid SSH prompts.

## 4. Create Inventory
```bash
echo "[target]" > inventory.ini
echo "$TARGET_HOST_IP" >> inventory.ini
```
Generates a dynamic Ansible inventory file with the target host IP.

## 5. Run Ansible Playbook
```bash
ansible-playbook -i inventory.ini playbook.yaml \
  --private-key ~/.ssh/id_rsa \
  -u ubuntu
```
Executes the Ansible playbook on the remote EC2 instance as the ubuntu user.

## 🚀 Usage
- Update the ```TARGET_HOST_IP``` in the pipeline or store it in Jenkins credentials.

- Replace:

  - ```https://your-repo-url.git``` → Your Git repository URL

  - ```playbook.yaml``` → Your Ansible playbook filename

  - ```ubuntu``` → Your EC2 SSH username if different

- Run the pipeline in Jenkins.

## 🔐 Security Notes
- Never hardcode private keys in the pipeline. Always use Jenkins credentials.

- Ensure ```.ssh``` permissions are correctly set:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
```
- Use ```ssh-keyscan``` to avoid interactive SSH prompts during automation.

