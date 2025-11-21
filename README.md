# ACIT 4640 Full Study Guide

## Table of Contents
1. [SSH (Secure Shell)](#section-1-ssh-secure-shell)
2. [Bash](#section-2-bash)
3. [systemd and systemctl](#section-3-systemd-and-systemctl)
4. [AWS CLI](#section-4-aws-cli)
5. [Infrastructure as Code (IaC)](#section-5-infrastructure-as-code-iac)
6. [Terraform](#section-6-terraform)
7. [Packer](#section-7-packer)
8. [Ansible](#section-8-ansible)
9. [Exam-Style Question Bank](#section-9-exam-style-question-bank)

---

## SECTION 1 — SSH (Secure Shell) — Expanded Conceptual Guide

### 1. What SSH Is (Concept Explanation)

SSH (Secure Shell) is a cryptographic network protocol used to securely access, manage, and communicate with remote systems. It's the standard tool for remotely administering Linux servers, network devices, cloud instances (like EC2), and anything requiring secure command-line access.

**Simple explanation:**
SSH is like opening a secure, encrypted tunnel between your computer and a remote machine so you can run commands safely.

**Why it's used:**
- You need to manage a remote machine
- You don't want anyone spying on your traffic
- You want identity-based authentication instead of passwords

**SSH protects against:**
- Password sniffing
- Session hijacking
- MITM attacks
- Packet tampering

### 2. Why SSH Matters (Exam-Relevant Understanding)

Your course uses SSH as the foundation for:
- Connecting to AWS EC2 Linux instances
- Ansible connections (Ansible uses SSH under the hood)
- Secure automation
- Handing keys, authentication, and remote commands
- Provisioning infrastructure via scripts

SSH is also crucial for:
- Git operations (SSH keys)
- Secure pipelines
- Remote execution (like heredocs into SSH sessions)

SSH shows up in almost every DevOps pipeline.

### 3. How SSH Works (The Connection Lifecycle)

Breaking down the SSH session establishment:

**Phase 1 — TCP Handshake**
- Client connects to server via TCP port 22.

**Phase 2 — Negotiation**
Client and server agree on:
- Encryption algorithm
- Hashing algorithm
- Compression method
- Key exchange method

**Phase 3 — Key Exchange**
- Uses asymmetric encryption (public/private keys) to securely establish a symmetric session key.
- SSH only uses asymmetric crypto temporarily — after the handshake, ALL data uses symmetric encryption for speed.

**Phase 4 — Authentication**
Options:
- Password
- Public key
- SSH agent
- Certificates
- Multi-factor (not common in your course)

**Phase 5 — Encrypted Session Established**
- Everything sent is encrypted, authenticated, and integrity-checked.

### 4. SSH Key Pairs (Deep Understanding)

**Public Key**
- Placed on the remote server
- Stored in `~/.ssh/authorized_keys` on the server
- Safe to share

**Private Key**
- Stays on the client
- Stored in `~/.ssh/id_ed25519` (or similar)
- Never shared

**Key Generation Command**
```bash
ssh-keygen -t ed25519 -f ~/.ssh/mykey -C "acit4640"
```

> **Exam trick:** RSA is older; ed25519 is recommended.

### 5. Important SSH Paths (Exam-Favorite Questions)

| Purpose | Path |
|---------|------|
| Private keys | `~/.ssh/id_ed25519` |
| Public keys | `~/.ssh/id_ed25519.pub` |
| SSH config | `~/.ssh/config` |
| Known hosts | `~/.ssh/known_hosts` |
| Authorized keys on server | `~/.ssh/authorized_keys` |

These appear in:
- Week 2 notes
- Readings 1

### 6. SSH Config File (Critical for Ansible & Automation)

**Location:** `~/.ssh/config`

**Example block:**
```
Host webserver
  HostName 54.23.22.10
  User ec2-user
  IdentityFile ~/.ssh/mykey
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

**Why important?**
- Avoids typing long ssh commands
- Lets Ansible automatically use the right identity
- Avoids interactive prompts in automation

### 7. SSH Options You Must Know

**Identity File**
```bash
ssh -i ~/.ssh/key.pem user@host
```

**Disable known_hosts prompts (for scripts)**
```bash
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null user@host
```
> **Exam trick:** This avoids messing your known_hosts file.

**Execute remote commands with heredocs**
```bash
ssh user@host << EOF
sudo yum install nginx -y
EOF
```

Used in automation scripts and Ansible labs.

### 8. Heredoc Through SSH (Common Testing Topic)

Local variables expand locally. Remote variables require escaping:

```bash
ssh user@host << EOF
echo "Local PWD: $PWD"
echo "Remote PWD: \$PWD"
EOF
```

This is from your Reading 1 (Heredocs).

### 9. SSH + Ansible Relationship

Ansible uses SSH to:
- Transfer modules
- Execute Python on remote hosts
- Retrieve results

**If SSH fails → Ansible fails.**

Your Week 7/11 notes reinforce this.

### 10. Common SSH Mistakes (Exam Trick Questions)

**Wrong permissions on private key**
If permissions are too open:
```bash
chmod 600 ~/.ssh/mykey
```

**Wrong user** (Ubuntu uses `ubuntu`, Amazon Linux uses `ec2-user`)

**Missing key on server**
Check: `~/.ssh/authorized_keys`

**Wrong inbound rule on AWS security group**
Port 22 must be allowed from your IP.

**Wrong file path**
Students often forget the SSH config lives in `~/.ssh/config`, not `/etc/ssh`.

### 11. Exam-Level Questions & Answers

**Q1: What is SSH and why is it secure?**
SSH is a protocol for secure remote access using encryption, key exchange, and hashing to protect communication.

**Q2: Why does SSH use both symmetric and asymmetric encryption?**
Asymmetric is used to securely exchange keys; symmetric is used for fast session encryption.

**Q3: What does StrictHostKeyChecking=no do?**
Prevents SSH from prompting when connecting to unknown hosts. Useful in automation.

**Q4: Where are SSH public keys stored on a server?**
`~/.ssh/authorized_keys`

**Q5: What file prevents duplicate known host entries?**
`~/.ssh/known_hosts`

**Q6: How does SSH relate to Ansible?**
Ansible uses SSH to push modules and execute commands on remote machines.

**Q7: What is the purpose of a private key?**
Authenticate the user without sending a password.

**Q8: What is a common cause of "Permission denied (publickey)" error?**
Wrong key file, wrong user, wrong permissions, or missing public key on server.

---

## SECTION 2 — Bash (Expanded Conceptual Guide)

### 1. What Bash Is (Concept Explanation)

Bash (Bourne Again SHell) is a command-line interpreter used on most Linux systems. It interprets commands, executes scripts, and automates tasks. It's the default shell in the ACIT 4640 labs and is crucial for:

- Writing automation scripts
- Creating infrastructure bootstrap scripts
- Passing variables into tools (Terraform, AWS CLI, Packer, Ansible)
- Running heredocs into SSH
- Command substitution for Terraform/Packer workflows

Understanding Bash is foundational for all DevOps.

### 2. Why Bash Matters (Exam Context)

Your course focuses heavily on automation. Bash is used to:
- Create folders
- Modify files
- Install packages
- Set environment variables for AWS CLI, Terraform, Packer
- Execute remote commands
- Build pipelines
- Manage services (systemctl)

**Exam questions often test:**
- Variable handling
- Environment propagation
- Execution order
- Here-doc behavior
- Quoting rules
- Script syntax

### 3. Variables (In-Depth)

**Definition:**
Variables store data (strings, numbers, commands, paths) used later in scripts.

**Create variable**
```bash
name="Hunter"
```

**Use variable**
```bash
echo "$name"
echo "${name}"
```
> `"${var}"` is preferred to avoid ambiguity.

**Rules**
- No spaces around `=`
- Must start with letter or underscore
- Case-sensitive

**Convention:**
- `lowercase`: script variables
- `UPPERCASE`: environment variables

### 4. Environment Variables

These variables are inherited by child processes (like Terraform or AWS CLI).

**Set variable for current session**
```bash
export AWS_PROFILE=acit4640_admin
```

**Environment variable examples:**
- `$HOME`
- `$PWD`
- `$USER`
- `$PATH`
- `$$` (current process ID)
- `$?` (exit code of last command)

**Exam Trick:**
Terraform loads credentials automatically from `~/.aws/credentials` via environment variables OR explicitly from CLI.

### 5. Command Substitution (VERY important)

Lets you store command output inside a variable.

```bash
today=$(date +%A)
```

**Useful for:**
- Naming backup folders
- Passing values to AWS CLI
- Determining state for Terraform provisioning
- Logging

**Example (from your readings)**
```bash
vpc_id=$(aws ec2 create-vpc ...)
```

### 6. Quoting Rules (Extremely Common Exam Topic)

**Double quotes (" ")**
Variables expand:
```bash
echo "Today is $today"
```

**Single quotes (' ')**
Variables do NOT expand:
```bash
echo '$today'   # literally prints $today
```

**No quotes**
Bash performs:
- globbing
- word splitting

DANGEROUS unless intentional.

### 7. Heredocs (High-value topic)

**What is a heredoc?**
A way to pass multiple lines of text into a command.

**Basic syntax**
```bash
cat << EOF
Hello $USER
EOF
```

**Quoted delimiter (no expansion)**
```bash
cat << 'EOF'
$USER will not expand
EOF
```

**Indented heredoc**
```bash
cat <<- EOF
	This uses tabs
EOF
```

**SSH heredoc (important!)**
```bash
ssh user@host << EOF
echo "Remote var: \$PWD"
EOF
```

Variables must be escaped (\$) to run on remote host.

### 8. Exit Codes

Every command returns an exit code:
- `0` = success
- Non-zero = failure

**Check exit code:**
```bash
$?
```

**Example:**
```bash
ls /fake
echo $?  # prints non-zero
```

**Use in scripts:**
```bash
if [ $? -eq 0 ]; then
  echo "Success"
fi
```

### 9. Conditionals (If Statements)

**Basic syntax:**
```bash
if [ condition ]; then
  # commands
fi
```

**With else:**
```bash
if [ "$name" = "Hunter" ]; then
  echo "Match"
else
  echo "No match"
fi
```

**File tests:**
- `-f file` = file exists
- `-d dir` = directory exists
- `-e path` = path exists

### 10. Loops

**For loop:**
```bash
for i in 1 2 3; do
  echo $i
done
```

**While loop:**
```bash
while [ condition ]; do
  # commands
done
```

**C-style for:**
```bash
for ((i=0; i<5; i++)); do
  echo $i
done
```

### 11. Functions

**Define function:**
```bash
my_function() {
  echo "Hello $1"
}
```

**Call function:**
```bash
my_function "Hunter"
```

**Return value:**
Functions return exit codes, not values. Use `echo` to output values.

### 12. Script Best Practices

**Shebang:**
```bash
#!/usr/bin/env bash
```

**Exit on error:**
```bash
set -e
```

**Debug mode:**
```bash
set -x
```

**Combined:**
```bash
set -ex
```

### 13. File Operations

**Read file:**
```bash
while read line; do
  echo "$line"
done < file.txt
```

**Write to file:**
```bash
echo "text" > file.txt   # overwrite
echo "text" >> file.txt  # append
```

**Redirect stderr:**
```bash
command 2> error.log
```

**Redirect both:**
```bash
command &> output.log
```

### 14. Common Bash Patterns

**Check if variable is set:**
```bash
if [ -z "$var" ]; then
  echo "Variable is empty"
fi
```

**Default value:**
```bash
name=${name:-"default"}
```

**Command exists check:**
```bash
if command -v aws &> /dev/null; then
  echo "AWS CLI installed"
fi
```

### 15. Exam Questions

**Q1: What does export do?**
Makes a variable available to child processes.

**Q2: What does $? represent?**
Exit code of last command.

**Q3: Difference between single vs double quotes?**
Single quotes: no variable expansion. Double quotes: variables expand.

**Q4: What is command substitution?**
Capturing output of a command: `var=$(command)`

**Q5: What file stores shell startup config?**
`~/.bashrc`

**Q6: How do you make a script executable?**
`chmod +x script.sh`

**Q7: What is a heredoc used for?**
Multi-line input into a command or SSH session.

**Q8: Why escape variables inside SSH heredocs?**
To avoid local expansion.

**Q9: What does set -e do in a script?**
Exit on any error.

**Q10: What symbol appends to a file?**
`>>`

---

## SECTION 3 — systemd and systemctl

### 1. What is systemd?

**systemd** is the init system and service manager for modern Linux distributions. It:
- Starts as PID 1
- Manages all services
- Handles dependencies
- Controls system state

**systemctl** is the command-line tool to interact with systemd.

### 2. Why systemd Matters

In your course, systemd is used to:
- Start/stop/enable services (nginx, docker, custom apps)
- Create custom service units
- Ensure services run at boot
- Manage application lifecycle in automation

### 3. Unit Files

**What is a unit file?**
A configuration file that tells systemd how to manage a service.

**Unit file locations:**
- System units: `/usr/lib/systemd/system/`
- Custom/modified units: `/etc/systemd/system/`

**Unit file structure:**
```ini
[Unit]
Description=My Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/myapp
Restart=always

[Install]
WantedBy=multi-user.target
```

### 4. Common systemctl Commands

**Start a service:**
```bash
systemctl start nginx
```

**Stop a service:**
```bash
systemctl stop nginx
```

**Restart a service:**
```bash
systemctl restart nginx
```

**Reload config (without stopping):**
```bash
systemctl reload nginx
```

**Enable at boot:**
```bash
systemctl enable nginx
```

**Disable at boot:**
```bash
systemctl disable nginx
```

**Check status:**
```bash
systemctl status nginx
```

**Check if service is active:**
```bash
systemctl is-active nginx
```

**Check if service is enabled:**
```bash
systemctl is-enabled nginx
```

### 5. Important systemd Concepts

**daemon-reload:**
After editing unit files, you must reload systemd:
```bash
systemctl daemon-reload
```

**Targets:**
Groups of units (like runlevels):
- `multi-user.target` = multi-user system (no GUI)
- `graphical.target` = multi-user with GUI

**Dependencies:**
- `After=` = start after this unit
- `Requires=` = requires this unit to run
- `Wants=` = soft dependency

### 6. Service Types

**Type=simple:**
Default. Process started by ExecStart is the main process.

**Type=forking:**
Service forks and parent exits.

**Type=oneshot:**
Runs once and exits.

**Type=notify:**
Service notifies systemd when ready.

### 7. Viewing Logs

**View service logs:**
```bash
journalctl -u nginx
```

**Follow logs:**
```bash
journalctl -u nginx -f
```

**Recent logs:**
```bash
journalctl -u nginx -n 50
```

**Since time:**
```bash
journalctl -u nginx --since "1 hour ago"
```

### 8. Common Troubleshooting

**Service won't start:**
1. Check status: `systemctl status service`
2. Check logs: `journalctl -u service`
3. Verify config: `systemctl show service`
4. Check dependencies

**Service fails at boot:**
- Check if enabled: `systemctl is-enabled service`
- Check ordering: Look at `After=` in unit file
- Check dependencies: Look at `Requires=` and `Wants=`

**Permission issues:**
- Unit files must be readable by root
- ExecStart path must be valid
- User= directive must reference valid user

### 9. Creating Custom Services

**Example: Web application service**
```ini
[Unit]
Description=My Web App
After=network.target

[Service]
Type=simple
User=webapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/start.sh
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

**Steps to deploy:**
1. Create file: `/etc/systemd/system/myapp.service`
2. Reload: `systemctl daemon-reload`
3. Enable: `systemctl enable myapp`
4. Start: `systemctl start myapp`
5. Check: `systemctl status myapp`

### 10. Exam Questions

**Q1: Where do system-level unit files live?**
`/usr/lib/systemd/system/`

**Q2: Where do modified/local unit files live?**
`/etc/systemd/system/`

**Q3: What does systemctl daemon-reload do?**
Reloads unit files after editing.

**Q4: Difference between restart vs reload?**
Restart = stop + start; Reload = re-read config without stopping.

**Q5: What is a systemd target?**
A group of units (like runlevels).

**Q6: How do you enable a service at boot?**
`systemctl enable nginx`

**Q7: What is PID 1?**
systemd

**Q8: How do you check failed units?**
`systemctl --failed`

**Q9: How do you view logs for a service?**
`journalctl -u nginx`

**Q10: What file specifies default boot target?**
`/etc/systemd/system/default.target`

---

## SECTION 4 — AWS CLI

### 1. What is AWS CLI?

The AWS Command Line Interface (CLI) is a unified tool to manage AWS services from the command line. It allows you to:
- Create/manage resources programmatically
- Automate AWS operations
- Script infrastructure deployment
- Query AWS APIs directly

**Why it matters in ACIT 4640:**
- Terraform, Packer, and Ansible all interact with AWS
- AWS CLI credentials are used by these tools
- Understanding AWS CLI helps debug infrastructure issues

### 2. Installation

**AWS CLI v2 (Required for course):**
Must use the official bundled installer.

**Linux installation:**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

**Verify installation:**
```bash
aws --version
```

### 3. Configuration

**AWS credentials location:**
`~/.aws/credentials`

**AWS config location:**
`~/.aws/config`

**Configure AWS CLI:**
```bash
aws configure
```

This prompts for:
- Access Key ID
- Secret Access Key
- Default region (use `us-west-2` for ACIT 4640)
- Default output format (`json`, `yaml`, `text`, or `table`)

### 4. Profiles

**Why profiles?**
Manage multiple AWS accounts or roles.

**Create named profile:**
```bash
aws configure --profile acit4640_admin
```

**Use profile in commands:**
```bash
aws s3 ls --profile acit4640_admin
```

**Set default profile:**
```bash
export AWS_PROFILE=acit4640_admin
```

**Import credentials from CSV:**
```bash
aws configure import --csv file://credentials.csv
```

### 5. Common AWS CLI Commands

**List S3 buckets:**
```bash
aws s3 ls
```

**Create S3 bucket:**
```bash
aws s3 mb s3://my-bucket-name
```

**List EC2 instances:**
```bash
aws ec2 describe-instances
```

**Create VPC:**
```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

**Get caller identity (verify credentials):**
```bash
aws sts get-caller-identity
```

### 6. Output Formats

**JSON (default):**
```bash
aws ec2 describe-instances --output json
```

**Table (human-readable):**
```bash
aws ec2 describe-instances --output table
```

**Text:**
```bash
aws ec2 describe-instances --output text
```

**YAML:**
```bash
aws ec2 describe-instances --output yaml
```

### 7. Querying with JMESPath

AWS CLI uses JMESPath to filter output.

**Example: Get instance IDs:**
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].InstanceId'
```

**Example: Get specific fields:**
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,InstanceType]'
```

**Filter by tag:**
```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=WebServer"
```

### 8. Environment Variables

**Set AWS credentials via environment:**
```bash
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
export AWS_DEFAULT_REGION=us-west-2
```

**Set profile:**
```bash
export AWS_PROFILE=acit4640_admin
```

**Important for scripts:**
These variables are inherited by Terraform, Packer, and Ansible.

### 9. AWS CLI + Terraform Integration

Terraform automatically uses:
1. Environment variables (`AWS_ACCESS_KEY_ID`, etc.)
2. Shared credentials file (`~/.aws/credentials`)
3. IAM role (if running on EC2)
4. Profile specified in provider block

**Example Terraform provider:**
```hcl
provider "aws" {
  region  = "us-west-2"
  profile = "acit4640_admin"
}
```

### 10. Common Issues

**Invalid credentials:**
- Check `~/.aws/credentials`
- Verify Access Key ID and Secret
- Check profile name

**Wrong region:**
- Verify `~/.aws/config`
- Or use `--region` flag
- Or set `AWS_DEFAULT_REGION`

**Permission denied:**
- IAM user needs appropriate policies
- Check policy attachments in AWS Console

**Command not found:**
- Verify installation: `which aws`
- Check PATH: `echo $PATH`

### 11. Exam Questions

**Q1: Where are AWS credentials stored?**
`~/.aws/credentials`

**Q2: Where are AWS profiles stored?**
`~/.aws/config`

**Q3: What region does ACIT 4640 use?**
`us-west-2`

**Q4: What installer must be used for AWS CLI v2?**
The official bundled installer (zip).

**Q5: How do you specify a profile in CLI commands?**
`--profile <name>`

**Q6: What does --query use?**
JMESPath

**Q7: How do you import a CSV file of IAM keys?**
`aws configure import --csv file://creds.csv`

**Q8: What is an Access Key ID?**
A programmatic credential.

**Q9: What is the default output format?**
`json`

**Q10: How to check AWS CLI version?**
`aws --version`

---

## SECTION 5 — Infrastructure as Code (IaC)

### 1. What is IaC?

**Infrastructure as Code (IaC)** is the practice of managing and provisioning infrastructure through machine-readable configuration files rather than manual processes.

**Simple explanation:**
Instead of clicking buttons in AWS Console, you write code that describes what you want, and tools build it for you.

### 2. Why IaC Matters

**Benefits:**
- **Consistency:** Same code = same infrastructure every time
- **Version Control:** Track changes in Git
- **Automation:** Eliminate manual errors
- **Documentation:** Code documents your infrastructure
- **Collaboration:** Teams work on same codebase
- **Speed:** Deploy faster than manual setup
- **Scalability:** Easily replicate environments

**Real-world use cases:**
- Spinning up dev/test/prod environments
- Disaster recovery
- Multi-region deployments
- Compliance and auditing

### 3. Declarative vs Imperative

**Declarative (WHAT you want):**
- You describe desired end state
- Tool figures out how to get there
- Examples: Terraform, CloudFormation

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
}
```

**Imperative (HOW to do it):**
- You specify exact steps
- You control the process
- Examples: Bash scripts, Ansible (mostly)

```bash
aws ec2 run-instances --image-id ami-12345 --instance-type t2.micro
```

### 4. Idempotency

**What is idempotency?**
Running the same code multiple times produces the same result.

**Why it matters:**
- Safe to re-run scripts
- No duplicate resources created
- Predictable outcomes

**Example:**
Terraform checks current state before making changes. If resource exists, it won't create a duplicate.

### 5. Immutable Infrastructure

**Traditional approach (mutable):**
- Create server
- Patch/update it over time
- Server "drifts" from original state

**Immutable approach:**
- Build new image with changes
- Deploy new instances
- Destroy old instances

**Benefits:**
- No configuration drift
- Consistent environments
- Easy rollbacks
- Better testing

### 6. Configuration Drift

**What is drift?**
When actual infrastructure differs from IaC definitions.

**Causes:**
- Manual changes in console
- Scripts run outside IaC
- Failed updates
- Time-based changes

**How to prevent:**
- Use IaC for ALL changes
- Regular state checks
- Automated compliance scanning
- Restrict console access

### 7. IaC Tool Categories

**Provisioning Tools:**
- **Terraform:** Multi-cloud provisioning
- **CloudFormation:** AWS-specific
- **Pulumi:** Code-based IaC

**Configuration Management:**
- **Ansible:** Agentless, uses SSH
- **Chef:** Agent-based
- **Puppet:** Agent-based

**Image Building:**
- **Packer:** Build machine images
- **Docker:** Container images

### 8. IaC in ACIT 4640

**Tools used:**
1. **Terraform:** Provision AWS infrastructure
2. **Packer:** Build custom AMIs
3. **Ansible:** Configure instances

**Workflow:**
1. Packer builds AMI with software pre-installed
2. Terraform provisions EC2 instances from AMI
3. Ansible handles additional configuration
4. Everything tracked in Git

### 9. Version Control Integration

**Why Git + IaC?**
- Track all infrastructure changes
- Code review for infrastructure
- Rollback to previous versions
- Collaboration workflow
- Audit trail

**Best practices:**
- Commit early and often
- Use meaningful commit messages
- Branch for experiments
- Tag releases
- Store state files securely (NOT in Git)

### 10. Common Patterns

**Environment separation:**
```
/terraform
  /dev
  /staging
  /prod
```

**Modular design:**
```
/modules
  /vpc
  /ec2
  /rds
```

**Shared variables:**
```
terraform.tfvars   # values
variables.tf       # definitions
```

### 11. Testing IaC

**Validation:**
- Terraform: `terraform validate`
- Packer: `packer validate`
- Ansible: `ansible-playbook --syntax-check`

**Plan/Dry-run:**
- Terraform: `terraform plan`
- Ansible: `--check` mode

**Automated testing:**
- Test in dev environment first
- Use CI/CD pipelines
- Automated compliance checks

### 12. Exam Questions

**Q1: What is IaC?**
Infrastructure defined and managed using machine-readable config files.

**Q2: 3 benefits of IaC:**
Consistency, Automation, Version control

**Q3: Declarative vs imperative IaC?**
Declarative = desired state. Imperative = step-by-step instructions.

**Q4: What is idempotency?**
Same result no matter how many times the code is run.

**Q5: What's immutable infrastructure?**
Servers replaced, not patched.

**Q6: What is drift?**
Actual infrastructure deviates from IaC definitions.

**Q7: What IaC tools are used in ACIT?**
Terraform, Packer, Ansible.

**Q8: What does version control add to IaC?**
History, collaboration, change tracking.

**Q9: Examples of IaC tool categories:**
Provisioning: Terraform; Config mgmt: Ansible; Image build: Packer

**Q10: Why does IaC improve reliability?**
Repeatable, predictable deployments.

---

## SECTION 6 — Terraform

### 1. What is Terraform?

**Terraform** is an open-source Infrastructure as Code tool by HashiCorp. It allows you to:
- Define infrastructure in code (HCL - HashiCorp Configuration Language)
- Provision resources across multiple cloud providers
- Manage infrastructure lifecycle
- Track infrastructure state

**Why Terraform in ACIT 4640:**
- Provision AWS resources (VPC, EC2, Security Groups, etc.)
- Declarative approach
- State management
- Dependency resolution

### 2. Terraform Workflow

**Standard workflow:**
```
init → plan → apply → destroy
```

**1. terraform init**
- Downloads provider plugins
- Initializes backend
- Downloads modules
- Creates `.terraform/` directory

```bash
terraform init
```

**2. terraform plan**
- Shows what changes will be made
- Validates configuration
- Compares desired state vs current state
- No changes applied

```bash
terraform plan
```

**3. terraform apply**
- Applies changes
- Creates/updates/deletes resources
- Updates state file
- Prompts for confirmation (unless `-auto-approve`)

```bash
terraform apply
```

**4. terraform destroy**
- Deletes all managed resources
- Cleans up infrastructure
- Updates state to empty

```bash
terraform destroy
```

### 3. Terraform State

**What is state?**
A snapshot of your infrastructure. Stored in `terraform.tfstate`.

**Why state matters:**
- Maps configuration to real-world resources
- Tracks metadata
- Improves performance
- Enables collaboration

**State file location:**
- Local: `terraform.tfstate`
- Remote: S3, Terraform Cloud, etc.

**Important:**
- Never edit state manually
- Never commit state to Git (contains secrets)
- Use remote state for teams

### 4. Terraform Files

**Main files:**
- `main.tf` - Primary configuration
- `variables.tf` - Input variable definitions
- `outputs.tf` - Output definitions
- `terraform.tfvars` - Variable values
- `provider.tf` - Provider configuration

**File extension:**
`.tf` for Terraform files

### 5. Providers

**What is a provider?**
A plugin that enables Terraform to interact with cloud platforms, SaaS, and other APIs.

**Example AWS provider:**
```hcl
provider "aws" {
  region  = "us-west-2"
  profile = "acit4640_admin"
}
```

**Common providers:**
- AWS
- Azure
- Google Cloud
- Docker
- GitHub

**Provider versions:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 6. Resources

**What is a resource?**
Infrastructure objects you want to create (EC2, VPC, S3, etc.)

**Syntax:**
```hcl
resource "resource_type" "name" {
  argument = "value"
}
```

**Example EC2 instance:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

**Referencing resources:**
```hcl
resource "aws_security_group" "web_sg" {
  name = "web-sg"
}

resource "aws_instance" "web" {
  ami             = "ami-12345"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.web_sg.id]
}
```

### 7. Variables

**Define variables:**
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

**Use variables:**
```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

**Variable types:**
- `string`
- `number`
- `bool`
- `list(type)`
- `map(type)`
- `object({...})`

**Set variable values:**
1. `terraform.tfvars` file
2. `-var` flag: `terraform apply -var="instance_type=t2.small"`
3. Environment variables: `TF_VAR_instance_type=t2.small`
4. Prompted input (if no default)

### 8. Outputs

**What are outputs?**
Values exported from Terraform that can be queried or used by other configurations.

**Define outputs:**
```hcl
output "instance_ip" {
  description = "Public IP of web server"
  value       = aws_instance.web.public_ip
}
```

**View outputs:**
```bash
terraform output
terraform output instance_ip
```

**Use in scripts:**
```bash
IP=$(terraform output -raw instance_ip)
```

### 9. Data Sources

**What is a data source?**
Reads information from existing resources (not created by Terraform).

**Example: Query existing AMI:**
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id
}
```

### 10. Modules

**What is a module?**
A reusable, self-contained package of Terraform configurations.

**Structure:**
```
modules/
  vpc/
    main.tf
    variables.tf
    outputs.tf
```

**Use module:**
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  name       = "my-vpc"
}
```

**Access module outputs:**
```hcl
resource "aws_instance" "web" {
  subnet_id = module.vpc.subnet_id
}
```

### 11. Terraform Commands

**Validate syntax:**
```bash
terraform validate
```

**Format code:**
```bash
terraform fmt
```

**Show current state:**
```bash
terraform show
```

**List resources:**
```bash
terraform state list
```

**Import existing resource:**
```bash
terraform import aws_instance.web i-1234567890abcdef0
```

**Refresh state:**
```bash
terraform refresh
```

### 12. Dependencies

**Implicit dependencies:**
Terraform automatically determines order based on resource references.

```hcl
resource "aws_instance" "web" {
  subnet_id = aws_subnet.main.id  # implicit dependency
}
```

**Explicit dependencies:**
Use `depends_on` when needed:

```hcl
resource "aws_instance" "web" {
  depends_on = [aws_s3_bucket.logs]
}
```

### 13. Common Patterns

**Conditional resources:**
```hcl
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
}
```

**Multiple similar resources:**
```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-${count.index}"
  }
}
```

**For_each:**
```hcl
variable "users" {
  default = ["alice", "bob", "charlie"]
}

resource "aws_iam_user" "example" {
  for_each = toset(var.users)
  name     = each.value
}
```

### 14. Best Practices

1. **Use version control** (Git)
2. **Use remote state** for teams
3. **Use modules** for reusability
4. **Use variables** for flexibility
5. **Use outputs** for important values
6. **Use workspaces** for environments
7. **Validate and format** before commit
8. **Plan before apply**
9. **Use .gitignore** for sensitive files
10. **Document your code**

### 15. Common Issues

**State lock errors:**
- Another process is using state
- Solution: Wait or remove lock manually

**Resource already exists:**
- Use `terraform import`
- Or rename in code

**Provider version conflicts:**
- Lock provider versions
- Run `terraform init -upgrade`

**Circular dependencies:**
- Refactor code
- Use data sources

### 16. Exam Questions

**Q1: Terraform workflow order?**
init → plan → apply → destroy

**Q2: Where is the state file stored?**
`terraform.tfstate`

**Q3: What does terraform init do?**
Downloads providers and modules.

**Q4: Why is state important?**
Maps real cloud resources to config.

**Q5: What is a provider?**
Plugin that integrates Terraform with AWS/Azure/GCP/etc.

**Q6: What is a module?**
Reusable block of Terraform config.

**Q7: How do you reference variables?**
`var.<name>`

**Q8: What is an output?**
Exported value from Terraform.

**Q9: What is a data source?**
Reads external info without creating resources.

**Q10: What folder stores providers?**
`.terraform/`

---

## SECTION 7 — Packer

### 1. What is Packer?

**Packer** is an open-source tool by HashiCorp for creating identical machine images across multiple platforms from a single source configuration.

**In simple terms:**
Packer automates the creation of custom AMIs (Amazon Machine Images) with pre-installed software and configuration.

**Why Packer matters:**
- **Faster deployments:** Software pre-installed in image
- **Consistency:** Same image = same configuration
- **Immutable infrastructure:** Don't patch, replace
- **Golden images:** Standardized, tested base images

### 2. Packer in ACIT 4640

**Workflow:**
1. Write Packer template (`.pkr.hcl`)
2. Packer builds temporary EC2 instance
3. Provisions software/config
4. Creates AMI snapshot
5. Terminates temporary instance
6. Terraform uses AMI to launch production instances

### 3. Packer Template Structure

**File extension:** `.pkr.hcl`

**Main components:**
1. **Source** - Base image and build environment
2. **Build** - Combines source and provisioners
3. **Provisioner** - Scripts/tools that configure the image

**Basic template structure:**
```hcl
packer {
  required_plugins {
    amazon = {
      source  = "github.com/hashicorp/amazon"
      version = "~> 1"
    }
  }
}

source "amazon-ebs" "ubuntu" {
  ami_name      = "my-custom-ami-{{timestamp}}"
  instance_type = "t2.micro"
  region        = "us-west-2"
  source_ami    = "ami-12345"
  ssh_username  = "ubuntu"
}

build {
  sources = ["source.amazon-ebs.ubuntu"]
  
  provisioner "shell" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
  }
}
```

### 4. Sources (Builders)

**What are sources?**
Define the base image and build environment.

**Common source types:**
- `amazon-ebs` - AWS AMI
- `docker` - Docker images
- `virtualbox-iso` - VirtualBox VMs
- `vmware-iso` - VMware VMs

**Example Amazon EBS source:**
```hcl
source "amazon-ebs" "ubuntu" {
  ami_name      = "webserver-{{timestamp}}"
  instance_type = "t2.micro"
  region        = "us-west-2"
  source_ami    = "ami-0abcdef1234567890"
  ssh_username  = "ubuntu"
  
  tags = {
    Name = "WebServer"
    OS   = "Ubuntu"
  }
}
```

**Key fields:**
- `ami_name` - Name of created AMI (must be unique)
- `source_ami` - Base AMI to start from
- `ssh_username` - User for SSH (ubuntu, ec2-user, etc.)
- `instance_type` - Temporary instance type
- `region` - AWS region

### 5. Provisioners

**What are provisioners?**
Steps that install software and configure the temporary instance.

**Common provisioner types:**
- `shell` - Run shell scripts
- `file` - Copy files to instance
- `ansible` - Run Ansible playbooks

**Shell provisioner (inline):**
```hcl
provisioner "shell" {
  inline = [
    "echo 'Installing nginx'",
    "sudo apt-get update",
    "sudo apt-get install -y nginx",
    "sudo systemctl enable nginx"
  ]
}
```

**Shell provisioner (script file):**
```hcl
provisioner "shell" {
  script = "scripts/install.sh"
}
```

**File provisioner:**
```hcl
provisioner "file" {
  source      = "config/nginx.conf"
  destination = "/tmp/nginx.conf"
}

provisioner "shell" {
  inline = [
    "sudo mv /tmp/nginx.conf /etc/nginx/nginx.conf"
  ]
}
```

### 6. Variables

**Define variables:**
```hcl
variable "ami_name" {
  type    = string
  default = "my-ami"
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

**Use variables:**
```hcl
source "amazon-ebs" "ubuntu" {
  ami_name      = var.ami_name
  instance_type = var.instance_type
}
```

**Set variable values:**
1. `.pkrvars.hcl` file
2. `-var` flag: `packer build -var="ami_name=my-custom-ami"`
3. Environment variables: `PKR_VAR_ami_name=my-custom-ami`

### 7. Packer Commands

**Validate template:**
```bash
packer validate template.pkr.hcl
```

**Format template:**
```bash
packer fmt template.pkr.hcl
```

**Build image:**
```bash
packer build template.pkr.hcl
```

**Build with variables:**
```bash
packer build -var="ami_name=custom" template.pkr.hcl
```

**Build specific source:**
```bash
packer build -only='amazon-ebs.ubuntu' template.pkr.hcl
```

### 8. Packer Installation

**Download binary:**
```bash
curl -O https://releases.hashicorp.com/packer/1.10.0/packer_1.10.0_linux_amd64.zip
```

**Extract:**
```bash
unzip packer_1.10.0_linux_amd64.zip
```

**Install:**
```bash
sudo mv packer /usr/local/bin/
```

**Verify:**
```bash
packer version
```

### 9. Communicators

**What are communicators?**
How Packer connects to the temporary instance.

**Types:**
- **SSH** (Linux/Unix)
- **WinRM** (Windows)

**SSH configuration:**
```hcl
source "amazon-ebs" "ubuntu" {
  ssh_username = "ubuntu"
  ssh_timeout  = "10m"
}
```

### 10. Packer + Terraform Integration

**Workflow:**
1. Packer builds AMI
2. Packer outputs AMI ID
3. Terraform references AMI ID
4. Terraform launches instances from AMI

**Packer output:**
```
==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs.ubuntu: AMIs were created:
us-west-2: ami-0abcdef1234567890
```

**Terraform data source:**
```hcl
data "aws_ami" "packer_image" {
  most_recent = true
  owners      = ["self"]
  
  filter {
    name   = "name"
    values = ["my-custom-ami-*"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.packer_image.id
}
```

### 11. Advanced Packer Features

**Multiple provisioners:**
```hcl
build {
  sources = ["source.amazon-ebs.ubuntu"]
  
  provisioner "file" {
    source      = "app/"
    destination = "/tmp/app"
  }
  
  provisioner "shell" {
    scripts = [
      "scripts/install-deps.sh",
      "scripts/configure-app.sh"
    ]
  }
  
  provisioner "shell" {
    inline = [
      "sudo systemctl enable myapp"
    ]
  }
}
```

**Post-processors:**
```hcl
post-processor "manifest" {
  output     = "manifest.json"
  strip_path = true
}
```

### 12. Debugging Packer

**Enable debug mode:**
```bash
PACKER_LOG=1 packer build template.pkr.hcl
```

**More verbose:**
```bash
PACKER_LOG=1 PACKER_LOG_PATH=packer.log packer build template.pkr.hcl
```

**Common issues:**
- SSH timeout: Increase `ssh_timeout`
- Security group issues: Check inbound rules
- IAM permissions: Verify packer can create instances
- AMI name conflicts: Use `{{timestamp}}` in name

### 13. Best Practices

1. **Use version control** for templates
2. **Use variables** for flexibility
3. **Use {{timestamp}}** in AMI names (ensures uniqueness)
4. **Tag your AMIs** with metadata
5. **Clean up old AMIs** periodically
6. **Test builds** in isolated environment
7. **Validate** before building
8. **Log everything** (PACKER_LOG)
9. **Use scripts** for complex provisioning
10. **Document** your build process

### 14. Example Complete Template

```hcl
packer {
  required_plugins {
    amazon = {
      source  = "github.com/hashicorp/amazon"
      version = "~> 1"
    }
  }
}

variable "ami_name" {
  type    = string
  default = "webserver"
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

source "amazon-ebs" "ubuntu" {
  ami_name      = "${var.ami_name}-{{timestamp}}"
  instance_type = var.instance_type
  region        = "us-west-2"
  source_ami    = "ami-0abcdef1234567890"
  ssh_username  = "ubuntu"
  
  tags = {
    Name        = var.ami_name
    Environment = "production"
    OS          = "Ubuntu 22.04"
  }
}

build {
  sources = ["source.amazon-ebs.ubuntu"]
  
  provisioner "file" {
    source      = "scripts/"
    destination = "/tmp/"
  }
  
  provisioner "shell" {
    inline = [
      "chmod +x /tmp/scripts/*.sh",
      "sudo /tmp/scripts/install.sh"
    ]
  }
  
  provisioner "shell" {
    script = "scripts/configure.sh"
  }
}
```

### 15. Exam Questions

**Q1: What does Packer do?**
Builds automated, reusable machine images.

**Q2: What file extension do Packer templates use?**
`.pkr.hcl`

**Q3: Where is the Packer binary usually installed?**
`/usr/local/bin/packer`

**Q4: What are builders?**
Components that define the base image + build environment.

**Q5: What are provisioners?**
Steps that configure the temporary VM.

**Q6: What are communicators?**
SSH or WinRM.

**Q7: What does Packer output?**
AMI ID (on AWS).

**Q8: Why are Packer images "golden"?**
Standardized, preconfigured, repeatable.

**Q9: How do you validate a template?**
`packer validate file.pkr.hcl`

**Q10: How to enable logs?**
`PACKER_LOG=1`

---

## SECTION 8 — Ansible

### 1. What is Ansible?

**Ansible** is an open-source automation tool for:
- Configuration management
- Application deployment
- Task automation
- Orchestration

**Key features:**
- **Agentless:** Uses SSH (no software on managed nodes)
- **Declarative:** Describe desired state
- **Idempotent:** Safe to run multiple times
- **Simple:** Uses YAML syntax
- **Powerful:** Extensive module library

### 2. Why Ansible Matters

In ACIT 4640, Ansible is used for:
- Configuring EC2 instances after Terraform provisions them
- Installing software packages
- Managing services
- Deploying applications
- Setting up systemd services

**Typical workflow:**
1. Packer builds base AMI
2. Terraform provisions infrastructure
3. Ansible configures running instances

### 3. Ansible Architecture

**Components:**
- **Control node:** Where Ansible runs (your machine)
- **Managed nodes:** Target servers
- **Inventory:** List of managed nodes
- **Modules:** Units of work (install package, start service, etc.)
- **Playbooks:** YAML files describing tasks
- **Roles:** Organized playbooks
- **Tasks:** Individual actions

**How Ansible works:**
1. Reads inventory
2. Connects via SSH
3. Copies Python modules to remote
4. Executes modules
5. Returns results
6. Removes temporary files

### 4. Inventory

**What is inventory?**
A file listing all managed hosts.

**Default location:**
`/etc/ansible/hosts`

**Local inventory:**
`inventory.ini` or `hosts.ini`

**INI format:**
```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/.ssh/mykey.pem
```

**YAML format:**
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    databases:
      hosts:
        db1.example.com:
  vars:
    ansible_user: ec2-user
    ansible_ssh_private_key_file: ~/.ssh/mykey.pem
```

**Important inventory variables:**
- `ansible_host` - IP address or hostname
- `ansible_user` - SSH username
- `ansible_ssh_private_key_file` - Path to SSH key
- `ansible_port` - SSH port (default 22)
- `ansible_python_interpreter` - Python path on remote

### 5. Ad-hoc Commands

**What are ad-hoc commands?**
Quick, one-time commands without playbooks.

**Syntax:**
```bash
ansible <hosts> -m <module> -a "<arguments>"
```

**Examples:**

**Ping all hosts:**
```bash
ansible all -m ping
```

**Check disk space:**
```bash
ansible webservers -m command -a "df -h"
```

**Install package:**
```bash
ansible webservers -m yum -a "name=nginx state=present" --become
```

**Restart service:**
```bash
ansible webservers -m service -a "name=nginx state=restarted" --become
```

**Copy file:**
```bash
ansible webservers -m copy -a "src=index.html dest=/var/www/html/"
```

### 6. Playbooks

**What is a playbook?**
A YAML file containing a list of plays. Each play targets hosts and runs tasks.

**Basic structure:**
```yaml
---
- name: Configure webservers
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
    
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

**Run playbook:**
```bash
ansible-playbook site.yml
```

**With specific inventory:**
```bash
ansible-playbook -i inventory.ini site.yml
```

**Check syntax:**
```bash
ansible-playbook --syntax-check site.yml
```

**Dry run:**
```bash
ansible-playbook --check site.yml
```

### 7. Tasks

**What is a task?**
A single action using a module.

**Task syntax:**
```yaml
- name: Task description
  module_name:
    parameter1: value1
    parameter2: value2
```

**Example tasks:**
```yaml
- name: Install package
  yum:
    name: httpd
    state: present

- name: Copy file
  copy:
    src: index.html
    dest: /var/www/html/index.html
    owner: apache
    mode: '0644'

- name: Create directory
  file:
    path: /opt/app
    state: directory
    mode: '0755'

- name: Run command
  command: /usr/local/bin/myscript.sh
  
- name: Start service
  service:
    name: httpd
    state: started
    enabled: yes
```

### 8. Modules

**What are modules?**
Reusable units of work. Ansible has 1000+ built-in modules.

**Common modules:**

**Package management:**
- `yum` - RedHat/CentOS
- `apt` - Debian/Ubuntu
- `package` - Generic (auto-detects)

**Service management:**
- `service` - Manage services
- `systemd` - systemd-specific control

**File operations:**
- `copy` - Copy files
- `file` - Create/delete/modify files/dirs
- `template` - Process Jinja2 templates

**Command execution:**
- `command` - Run commands
- `shell` - Run shell commands
- `script` - Run local script on remote

**Cloud:**
- `ec2` - Manage EC2 instances
- `s3_bucket` - Manage S3 buckets

### 9. Variables

**Define variables in playbook:**
```yaml
- name: Deploy app
  hosts: webservers
  vars:
    app_name: myapp
    app_port: 8080
  
  tasks:
    - name: Create app directory
      file:
        path: "/opt/{{ app_name }}"
        state: directory
```

**Variables file:**
```yaml
# vars.yml
app_name: myapp
app_port: 8080
db_host: db.example.com
```

**Use variables file:**
```yaml
- name: Deploy app
  hosts: webservers
  vars_files:
    - vars.yml
  
  tasks:
    - name: Configure app
      template:
        src: config.j2
        dest: "/opt/{{ app_name }}/config.ini"
```

**Variable precedence (highest to lowest):**
1. Extra vars (`-e` flag)
2. Task vars
3. Play vars
4. Host vars
5. Group vars
6. Defaults

### 10. Templates (Jinja2)

**What are templates?**
Dynamic configuration files using Jinja2 syntax.

**Template file (config.j2):**
```jinja2
server {
    listen {{ app_port }};
    server_name {{ ansible_hostname }};
    
    location / {
        proxy_pass http://{{ app_host }}:{{ app_port }};
    }
}
```

**Use template:**
```yaml
- name: Deploy nginx config
  template:
    src: config.j2
    dest: /etc/nginx/sites-available/myapp
  notify: Restart nginx
```

**Jinja2 features:**
- Variables: `{{ variable }}`
- Conditionals: `{% if condition %}`
- Loops: `{% for item in list %}`
- Filters: `{{ variable | upper }}`

### 11. Handlers

**What are handlers?**
Special tasks that run only when notified by other tasks.

**Use case:** Restart service only if config changed.

**Define handler:**
```yaml
handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

**Notify handler:**
```yaml
tasks:
  - name: Copy nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx
```

**Key points:**
- Handlers run at end of play
- Handlers run once (even if notified multiple times)
- Handlers run in order defined

### 12. Roles

**What are roles?**
A way to organize playbooks into reusable components.

**Role structure:**
```
roles/
  webserver/
    tasks/
      main.yml
    handlers/
      main.yml
    templates/
      nginx.conf.j2
    files/
      index.html
    vars/
      main.yml
    defaults/
      main.yml
```

**Use role:**
```yaml
- name: Configure servers
  hosts: webservers
  roles:
    - webserver
    - firewall
```

**Create role:**
```bash
ansible-galaxy init webserver
```

### 13. Privilege Escalation

**become:**
Runs tasks with elevated privileges (sudo).

**Enable for entire play:**
```yaml
- name: Install software
  hosts: all
  become: yes
  
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
```

**Enable for specific task:**
```yaml
tasks:
  - name: Install package
    yum:
      name: nginx
      state: present
    become: yes
```

**Become method:**
```yaml
become_method: sudo
become_user: root
```

### 14. Conditionals

**Run task based on condition:**
```yaml
- name: Install apache (RedHat)
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"

- name: Install apache (Debian)
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"
```

**Multiple conditions:**
```yaml
when:
  - ansible_distribution == "Ubuntu"
  - ansible_distribution_version == "22.04"
```

### 15. Loops

**Loop over list:**
```yaml
- name: Install packages
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - vim
```

**Loop with complex data:**
```yaml
- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'alice', groups: 'admin' }
    - { name: 'bob', groups: 'users' }
```

### 16. Ansible Configuration

**Config file locations (precedence):**
1. `ANSIBLE_CONFIG` environment variable
2. `ansible.cfg` in current directory
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

**Example ansible.cfg:**
```ini
[defaults]
inventory = ./inventory.ini
private_key_file = ~/.ssh/mykey.pem
host_key_checking = False
retry_files_enabled = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

### 17. SSH + Ansible

**How Ansible uses SSH:**
- Connects via SSH to each managed node
- Requires SSH access (key or password)
- Uses SSH keys stored in `~/.ssh/`

**If SSH fails → Ansible fails**

**Ways Ansible interacts with SSH:**
- Identifies host via `ansible_host`
- Uses `ansible_user`
- Uses `ansible_ssh_private_key_file`
- Uses Python on remote for modules

### 18. Common Problems (Exam-Level Troubleshooting)

**Missing Python on remote:**
```
UNREACHABLE! => 'python not found'
```
Solution: Install python manually.

**SSH key path is wrong:**
```
Permission denied (publickey)
```
Solution: Fix key file or inventory.

**Host key checking prompt:**
Set `host_key_checking=False` in `ansible.cfg`

**YAML indentation errors:**
```
ERROR! mapping values are not allowed here
```
Fix spaces, not tabs.

### 19. Ansible Important Paths

| Purpose | Path |
|---------|------|
| Inventory | `/etc/ansible/hosts` or local `inventory.ini` |
| Config file | `/etc/ansible/ansible.cfg` or local `ansible.cfg` |
| SSH keys | `~/.ssh/id_ed25519` |
| Python path on remote | `/usr/bin/python` |
| Roles directory | `roles/` |

### 20. Example Exam Questions (with clear answers)

**Q1: What is Ansible and why is it agentless?**
Ansible is a configuration management tool that uses SSH instead of agents.

**Q2: Where is the default inventory file located?**
`/etc/ansible/hosts`

**Q3: What does Ansible use to connect to remote hosts?**
SSH + Python.

**Q4: What is idempotency in Ansible?**
Running the same playbook gives the same result without duplicate actions.

**Q5: What is a handler?**
A special task triggered only when notified by other tasks.

**Q6: How do you run an ad-hoc command?**
`ansible all -m ping`

**Q7: What is a Jinja2 template used for?**
Dynamic config file generation.

**Q8: Name Ansible's main building blocks.**
Inventory, modules, playbooks, tasks, handlers, roles, templates.

**Q9: What does become: yes do?**
Enables privilege escalation (sudo).

**Q10: What command runs a playbook?**
`ansible-playbook site.yml`

---

## SECTION 9 — Exam-Style Question Bank (Master List)

This is a comprehensive set of exam-style questions covering all sections. Use these as practice and self-testing.

Questions are grouped by topic, with short, clear answers you can memorize.

### SECTION 9A — SSH Exam Questions

1. **What port does SSH use?**  
   22

2. **Where are SSH private keys stored?**  
   `~/.ssh/id_ed25519` (or `id_rsa`)

3. **Where does SSH store known hosts?**  
   `~/.ssh/known_hosts`

4. **Where are public keys stored on a server?**  
   `~/.ssh/authorized_keys`

5. **What does StrictHostKeyChecking=no do?**  
   Disables host key verification (used in automation).

6. **What cryptography does SSH use?**  
   Asymmetric (for handshake) → Symmetric (for session)

7. **How do you specify a key file when connecting?**  
   `ssh -i ~/.ssh/key.pem user@host`

8. **What is the SSH config file path?**  
   `~/.ssh/config`

9. **What causes "Permission denied (publickey)"?**  
   Wrong key, wrong perms, wrong user, wrong host entry.

10. **Difference between local and remote variables in SSH heredocs?**  
    Local expand immediately; remote must be escaped with `\$`.

### SECTION 9B — Bash Exam Questions

1. **What does export do?**  
   Makes a variable available to child processes.

2. **What does $? represent?**  
   Exit code of last command.

3. **What's the difference between single vs double quotes?**  
   Single quotes: no variable expansion. Double quotes: variables expand.

4. **What is command substitution?**  
   Capturing output of a command: `var=$(command)`

5. **What file stores shell startup config?**  
   `~/.bashrc`

6. **How do you make a script executable?**  
   `chmod +x script.sh`

7. **What is a heredoc used for?**  
   Multi-line input into a command or SSH session.

8. **Why escape variables inside SSH heredocs?**  
   To avoid local expansion.

9. **What does set -e do in a script?**  
   Exit on any error.

10. **What symbol appends to a file?**  
    `>>`

### SECTION 9C — systemd / systemctl Exam Questions

1. **Where do system-level unit files live?**  
   `/usr/lib/systemd/system/`

2. **Where do modified/local unit files live?**  
   `/etc/systemd/system/`

3. **What does systemctl daemon-reload do?**  
   Reloads unit files after editing.

4. **Difference between restart vs reload?**  
   Restart = stop + start; Reload = re-read config without stopping.

5. **What is a systemd target?**  
   A group of units (like runlevels).

6. **How do you enable a service at boot?**  
   `systemctl enable nginx`

7. **What is PID 1?**  
   systemd

8. **How do you check failed units?**  
   `systemctl --failed`

9. **How do you view logs for a service?**  
   `journalctl -u nginx`

10. **What file specifies default boot target?**  
    `/etc/systemd/system/default.target`

### SECTION 9D — AWS CLI Exam Questions

1. **Where are AWS credentials stored?**  
   `~/.aws/credentials`

2. **Where are AWS profiles stored?**  
   `~/.aws/config`

3. **What region does ACIT 4640 use?**  
   `us-west-2`

4. **What installer must be used for AWS CLI v2?**  
   The official bundled installer (zip).

5. **How do you specify a profile in CLI commands?**  
   `--profile <name>`

6. **What does --query use?**  
   JMESPath

7. **How do you import a CSV file of IAM keys?**  
   `aws configure import --csv file://creds.csv`

8. **What is an Access Key ID?**  
   A programmatic credential.

9. **What is the default output format?**  
   `json`

10. **How to check AWS CLI version?**  
    `aws --version`

### SECTION 9E — Infrastructure as Code (IaC) Exam Questions

1. **What is IaC?**  
   Infrastructure defined and managed using machine-readable config files.

2. **3 benefits of IaC:**  
   Consistency, Automation, Version control

3. **Declarative vs imperative IaC?**  
   Declarative = desired state. Imperative = step-by-step instructions.

4. **What is idempotency?**  
   Same result no matter how many times the code is run.

5. **What's immutable infrastructure?**  
   Servers replaced, not patched.

6. **What is drift?**  
   Actual infrastructure deviates from IaC definitions.

7. **What IaC tools are used in ACIT?**  
   Terraform, Packer, Ansible.

8. **What does version control add to IaC?**  
   History, collaboration, change tracking.

9. **Examples of IaC tool categories:**  
   Provisioning: Terraform; Config mgmt: Ansible; Image build: Packer

10. **Why does IaC improve reliability?**  
    Repeatable, predictable deployments.

### SECTION 9F — Terraform Exam Questions

1. **Terraform workflow order?**  
   init → plan → apply → destroy

2. **Where is the state file stored?**  
   `terraform.tfstate`

3. **What does terraform init do?**  
   Downloads providers and modules.

4. **Why is state important?**  
   Maps real cloud resources to config.

5. **What is a provider?**  
   Plugin that integrates Terraform with AWS/Azure/GCP/etc.

6. **What is a module?**  
   Reusable block of Terraform config.

7. **How do you reference variables?**  
   `var.<name>`

8. **What is an output?**  
   Exported value from Terraform.

9. **What is a data source?**  
   Reads external info without creating resources.

10. **What folder stores providers?**  
    `.terraform/`

### SECTION 9G — Packer Exam Questions

1. **What does Packer do?**  
   Builds automated, reusable machine images.

2. **What file extension do Packer templates use?**  
   `.pkr.hcl`

3. **Where is the Packer binary usually installed?**  
   `/usr/local/bin/packer`

4. **What are builders?**  
   Components that define the base image + build environment.

5. **What are provisioners?**  
   Steps that configure the temporary VM.

6. **What are communicators?**  
   SSH or WinRM.

7. **What does Packer output?**  
   AMI ID (on AWS).

8. **Why are Packer images "golden"?**  
   Standardized, preconfigured, repeatable.

9. **How do you validate a template?**  
   `packer validate file.pkr.hcl`

10. **How to enable logs?**  
    `PACKER_LOG=1`

### SECTION 9H — Ansible Exam Questions

1. **What does Ansible use to communicate?**  
   SSH (and Python on remote)

2. **Where is the default inventory stored?**  
   `/etc/ansible/hosts`

3. **What command runs a playbook?**  
   `ansible-playbook site.yml`

4. **What is a handler?**  
   Runs only when notified.

5. **What is idempotency in Ansible?**  
   Repeated runs produce same state.

6. **What is a module?**  
   Reusable block that performs an action (e.g., yum, service).

7. **What is a role?**  
   Structured way to organize playbooks into reusable components.

8. **What does become: yes do?**  
   Privilege escalation (sudo).

9. **What is a Jinja2 template?**  
   Dynamic config file using variables.

10. **Why does Ansible not require agents?**  
    It uses SSH only.

---
