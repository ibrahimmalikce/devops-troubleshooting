# GitHub Actions Self-Hosted Runner Setup on AlmaLinux 9

## Environment
- **OS:** AlmaLinux 9
- **SSH Port:** 4307

## Step 1: Create User Account
\`\`\`bash
adduser malik
usermod -aG wheel malik
passwd malik
\`\`\`

## Step 2: Disable Root Login
\`\`\`bash
nano /etc/ssh/sshd_config
# Set: PermitRootLogin no
systemctl restart sshd
\`\`\`

### Issue: PermitRootLogin still active after change
\`\`\`bash
grep -r "PermitRootLogin" /etc/ssh/
nano /etc/ssh/sshd_config.d/01-permitrootlogin.conf
# Set: PermitRootLogin no
systemctl restart sshd
sshd -T | grep permitrootlogin
\`\`\`

## Step 3: Setup GitHub Actions Runner
\`\`\`bash
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.335.1.tar.gz -L \
https://github.com/actions/runner/releases/download/v2.335.1/actions-runner-linux-x64-2.335.1.tar.gz
tar xzf ./actions-runner-linux-x64-2.335.1.tar.gz
./config.sh --url <REPO_URL> --token <TOKEN>
\`\`\`

## Step 4: Install as Permanent Service
\`\`\`bash
sudo ./svc.sh install
sudo ./svc.sh start
\`\`\`
