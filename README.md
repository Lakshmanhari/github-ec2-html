Yes. If you have **only an `index.html` file** in GitHub and want to deploy it to an **EC2 Ubuntu server using GitHub Actions**, you can keep it very simple.

### Architecture

```text
GitHub Repository
       │
       │ git push
       ▼
GitHub Actions
       │
       │ SSH
       ▼
EC2 Ubuntu Server
       │
       ▼
/var/www/html/index.html
       │
       ▼
     Nginx
       │
       ▼
   Browser
```

## 1. EC2 setup

Connect to your EC2 instance:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Install Nginx:

```bash
sudo apt update
sudo apt install nginx -y
```

Check:

```bash
sudo systemctl status nginx
```

Your default website directory is:

```text
/var/www/html/
```

Test:

```bash
echo "<h1>Hello from EC2</h1>" | sudo tee /var/www/html/index.html
```

Then open:

```text
http://YOUR_EC2_PUBLIC_IP
```

Make sure the EC2 **Security Group** allows:

```text
HTTP   TCP   80   0.0.0.0/0
```

---

# 2. GitHub repository

Your repository can simply contain:

```text
my-website/
├── index.html
└── .github/
    └── workflows/
        └── deploy.yml
```

Example `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>
<body>
    <h1>Hello from GitHub Actions!</h1>
    <p>Deployed automatically to EC2.</p>
</body>
</html>
```

---

# 3. Create an SSH key for GitHub Actions

You need a key that GitHub Actions can use to SSH into EC2.

On your local machine:

```bash
ssh-keygen -t ed25519 -C "github-actions"
```

You'll get:

```text
id_ed25519
id_ed25519.pub
```

Copy the **public key**:

```bash
cat ~/.ssh/id_ed25519.pub
```

On the EC2 server:

```bash
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```

Paste the public key and save.

Then:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

# 4. Add GitHub Secrets

Go to:

**GitHub → Repository → Settings → Secrets and variables → Actions**

Create these secrets:

| Secret        | Value                            |
| ------------- | -------------------------------- |
| `EC2_HOST`    | EC2 public IP                    |
| `EC2_USER`    | `ubuntu`                         |
| `EC2_SSH_KEY` | Contents of private `id_ed25519` |

For `EC2_SSH_KEY`, copy the **entire private key**, including:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

**Do not upload the private key to your repository.**

---

# 5. GitHub Actions workflow

Create:

```text
.github/workflows/deploy.yml
```

Use:

```yaml
name: Deploy HTML to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to EC2
        uses: appleboy/scp-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "index.html"
          target: "/tmp"

      - name: Move file to Nginx directory
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo cp /tmp/index.html /var/www/html/index.html
            sudo systemctl reload nginx
```

Now:

```bash
git add .
git commit -m "Deploy website"
git push origin main
```

GitHub Actions automatically runs.

### What happens?

```text
git push
   ↓
GitHub Actions starts
   ↓
Checkout index.html
   ↓
SCP index.html → EC2 /tmp
   ↓
SSH into EC2
   ↓
sudo cp → /var/www/html/index.html
   ↓
reload nginx
   ↓
Website updated
```

Then open:

```text
http://YOUR_EC2_PUBLIC_IP
```

---

### Even simpler approach

Since this is just **one HTML file**, you don't need Docker, Jenkins, ECR, Kubernetes, Terraform, etc.

The important DevOps concepts you're demonstrating are:

**GitHub → GitHub Actions → SSH/SCP → EC2 → Nginx → Website**

This is actually a good small project to understand **CI/CD deployment to EC2** before moving to Docker-based deployments.

-----------------------------------------------------------
this is a step by step 
Sure. Let's do it **one step at a time**, from zero. We'll deploy only `index.html` from GitHub → EC2 using **GitHub Actions + Nginx**.

Don't move to the next step until the current one works.

## Step 1 — Create the HTML file

On your Windows PC, create a folder:

```text
github-ec2-html
```

Inside it, create:

```text
index.html
```

Put this inside:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My EC2 Website</title>
</head>
<body>
    <h1>Hello from EC2!</h1>
    <p>Deployed using GitHub Actions.</p>
</body>
</html>
```

Your folder should look like:

```text
github-ec2-html
└── index.html
```

---

## Step 2 — Create GitHub repository

Go to [GitHub](https://github.com?utm_source=chatgpt.com).

Create a **new repository**.

For example:

```text
github-ec2-html
```

Choose:

* Repository: `github-ec2-html`
* Public or Private: either is fine
* Don't add README for now

Click **Create repository**.

---

## Step 3 — Push your HTML file to GitHub

Open PowerShell inside your folder.

For example:

```powershell
cd "C:\Users\Lakshman Hari\Desktop\github-ec2-html"
```

Then:

```powershell
git init
```

```powershell
git add .
```

```powershell
git commit -m "Initial HTML website"
```

Set your branch:

```powershell
git branch -M main
```

Connect your GitHub repository:

```powershell
git remote add origin https://github.com/YOUR_USERNAME/github-ec2-html.git
```

Then:

```powershell
git push -u origin main
```

Go to GitHub and confirm you can see:

```text
index.html
```

---

# Step 4 — Prepare your EC2 server

Now connect to your Ubuntu EC2:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Example:

```bash
ssh -i myserver.pem ubuntu@13.234.XX.XX
```

Once connected, run:

```bash
sudo apt update
```

Then install Nginx:

```bash
sudo apt install nginx -y
```

---

# Step 5 — Start Nginx

Run:

```bash
sudo systemctl start nginx
```

Then:

```bash
sudo systemctl enable nginx
```

Check:

```bash
sudo systemctl status nginx
```

You should see something similar to:

```text
Active: active (running)
```

Press:

```text
q
```

to exit.

---

# Step 6 — Open port 80

Go to:

**AWS Console → EC2 → Instances → Your Instance → Security**

Click the **Security Group**.

Go to:

**Inbound rules → Edit inbound rules**

Add:

| Type | Port | Source    |
| ---- | ---: | --------- |
| HTTP |   80 | 0.0.0.0/0 |

Save.

Now open:

```text
http://YOUR_EC2_PUBLIC_IP
```

You should see the **Nginx welcome page**.

If you see the Nginx page, your EC2 server is ready.

---

# Step 7 — Create SSH key for GitHub Actions

This is the important part.

On **your Windows PC**, open PowerShell.

Run:

```powershell
ssh-keygen -t ed25519 -C "github-actions"
```

It will ask:

```text
Enter file in which to save the key:
```

Press **Enter**.

Then:

```text
Enter passphrase:
```

Press **Enter**.

Again:

```text
Enter same passphrase again:
```

Press **Enter**.

You should now have:

```text
C:\Users\Lakshman Hari\.ssh\id_ed25519
C:\Users\Lakshman Hari\.ssh\id_ed25519.pub
```

There are two keys:

```text
id_ed25519       ← PRIVATE KEY
id_ed25519.pub   ← PUBLIC KEY
```

**Never upload `id_ed25519` to GitHub.**

---

# Step 8 — Put the public key into EC2

On Windows, run:

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
```

You'll get something like:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... github-actions
```

Copy the **entire line**.

Now SSH into EC2:

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Run:

```bash
mkdir -p ~/.ssh
```

Then:

```bash
nano ~/.ssh/authorized_keys
```

Paste the GitHub Actions public key at the bottom.

Save:

```text
Ctrl + O
Enter
Ctrl + X
```

Then:

```bash
chmod 700 ~/.ssh
```

```bash
chmod 600 ~/.ssh/authorized_keys
```

---

# Step 9 — Test the new SSH key

This is important before creating GitHub Actions.

On your Windows PC, run:

```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" ubuntu@YOUR_EC2_PUBLIC_IP
```

If it connects to EC2 without asking for your `.pem` file, the SSH key is working.

You should see:

```text
ubuntu@ip-xxx-xx-xx-xx:~$
```

Exit:

```bash
exit
```

---

# Step 10 — Add GitHub Secrets

Go to your GitHub repository.

Then:

**Settings → Secrets and variables → Actions**

Click:

**New repository secret**

Create these **three secrets**.

### Secret 1

Name:

```text
EC2_HOST
```

Value:

```text
YOUR_EC2_PUBLIC_IP
```

Example:

```text
13.234.XX.XX
```

---

### Secret 2

Name:

```text
EC2_USER
```

Value:

```text
ubuntu
```

---

### Secret 3

Name:

```text
EC2_SSH_KEY
```

On your Windows PC run:

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519"
```

You'll see:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
...
...
-----END OPENSSH PRIVATE KEY-----
```

Copy **everything**.

Paste it as the value of:

```text
EC2_SSH_KEY
```

So GitHub should have:

```text
EC2_HOST
EC2_USER
EC2_SSH_KEY
```

---

# Step 11 — Create GitHub Actions workflow

Now go back to your project.

Create this folder:

```text
.github
```

Inside:

```text
.github/workflows
```

Inside that create:

```text
deploy.yml
```

Your project now looks like:

```text
github-ec2-html/
│
├── index.html
│
└── .github/
    └── workflows/
        └── deploy.yml
```

Put this inside `deploy.yml`:

```yaml
name: Deploy HTML to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Copy HTML to EC2
        uses: appleboy/scp-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "index.html"
          target: "/tmp"

      - name: Deploy HTML
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo cp /tmp/index.html /var/www/html/index.html
            sudo systemctl reload nginx
```

---

# Step 12 — Push the workflow

From your project directory:

```powershell
git add .
```

Then:

```powershell
git commit -m "Add GitHub Actions deployment"
```

Then:

```powershell
git push
```

---

# Step 13 — Check GitHub Actions

Go to your GitHub repository.

Click:

**Actions**

You should see:

```text
Deploy HTML to EC2
```

Click it.

You should see:

```text
✓ Checkout code
✓ Copy HTML to EC2
✓ Deploy HTML
```

If all three are green:

**Your deployment is successful.**

---

# Step 14 — Test the website

Open:

```text
http://YOUR_EC2_PUBLIC_IP
```

You should now see:

```text
Hello from EC2!

Deployed using GitHub Actions.
```

🎉

---

# Step 15 — Test automatic deployment

Now change your `index.html`:

```html
<h1>Hello from my GitHub Actions CI/CD!</h1>
```

Then:

```powershell
git add .
git commit -m "Update website"
git push
```

GitHub Actions automatically runs again:

```text
GitHub
   ↓
Push
   ↓
GitHub Actions
   ↓
Copy index.html
   ↓
EC2
   ↓
/var/www/html/index.html
   ↓
Nginx
   ↓
Website updated
```

Refresh the browser and you'll see the new content.

---

## The complete setup you're learning

```text
             GitHub
                │
                │ git push
                ▼
       ┌─────────────────┐
       │ GitHub Actions  │
       └────────┬────────┘
                │
             SSH/SCP
                │
                ▼
       ┌─────────────────┐
       │   EC2 Ubuntu    │
       │                 │
       │ /var/www/html/  │
       │    index.html   │
       └────────┬────────┘
                │
              Nginx
                │
                ▼
             Browser
```

**For now, start only with Steps 1–6.** Once your EC2 shows the Nginx welcome page, tell me **“Step 6 done”**, and I'll take you through **Step 7 (SSH key)** one command at a time.

