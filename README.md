# Node App — Jenkins Freestyle Deployment (Step-by-step)

> This README describes how to deploy a simple Node.js application using **Jenkins Freestyle** jobs. It includes setup for Node.js / npm / PM2 for the `jenkins` user, three Jenkins jobs (pull, install deps, deploy), GitHub webhook trigger, and where to add screenshots.

---

## 🧾 Overview

We will create three Jenkins freestyle jobs:

1. `node-pull-repo` — pulls code from GitHub (no build steps).
2. `node-install-deps` — runs `npm install` in the pulled workspace.
3. `node-deploy-app` — starts/restarts the app using PM2.

Jenkins will run commands as the `jenkins` user. We'll install Node.js, npm, and PM2 globally for that user so `sudo` is not required inside Jenkins jobs.

---

## 📁 Repository

Use this repo URL (example used in this guide):

```
https://github.com/iamtruptimane/node-js-app-CICD.git
```

Replace with your repo URL and branch as needed.

---

## ⚙️ Prerequisites (on server)

Run the following on the server (as a user with `sudo` privileges) to prepare the environment for the `jenkins` user:

```bash
# Update package lists (optional but recommended)
sudo apt update

# Install Node.js
sudo apt install -y nodejs

# Install npm
sudo apt install -y npm

# Install PM2 globally (for jenkins user) — run as the jenkins user or install globally
# If you run as root, PM2 will be owned by root. To make PM2 available to jenkins, either:
# 1) run the following as the 'jenkins' user (recommended), or
# 2) change npm global prefix for jenkins.

# Switch to jenkins user and install pm2 globally
sudo su - jenkins -s /bin/bash -c "npm install -g pm2"

# Verify installations (as jenkins)
sudo su - jenkins -s /bin/bash -c "node -v && npm -v && pm2 -v"
```

> **Notes:**
>
> * Installing PM2 as the `jenkins` user ensures Jenkins can run `pm2 start` without `sudo`.
> * If your system requires a specific Node.js version, consider using NodeSource or `nvm` for version control.

---

## 🔐 Permissions & Paths

This guide assumes Jenkins workspace is located at:

```
/var/lib/jenkins/workspace/node-pull-repo
```

Make sure the `jenkins` user owns the workspace and has permission to run `npm` and `pm2`:

```bash
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace
```

If your Jenkins home is elsewhere, update the `cd` paths in the job shell steps accordingly.

---

## 🧱 Job 1 — node-pull-repo (Pull repository)

1. Jenkins → **New Item** → **Freestyle project** → Name: `node-pull-repo`
2. **Source Code Management** → **Git** → Repository URL: `https://github.com/iamtruptimane/node-js-app-CICD.git`
3. **Branch Specifier**: `main` (or your default branch)
4. **Build Triggers**: Check **GitHub hook trigger for GITScm polling** (to enable webhook-triggered builds)
5. **Build**: Leave **Build Steps** empty (job only pulls code).
6. Click **Save**.

**Screenshot placeholder:**

```
![Screenshot: Create Job - node-pull-repo](screenshots/jenkins-create-node-pull-repo.png)
```

---

## 🧩 Job 2 — node-install-deps (Install dependencies)

1. Jenkins → **New Item** → **Freestyle project** → Name: `node-install-deps`
2. **Build Triggers** → **Build after other projects are built** → Enter `node-pull-repo` (so it runs after pulling)
3. **Build Steps** → **Execute shell** → Script:

```bash
# Navigate to the pulled repository workspace
cd "/var/lib/jenkins/workspace/node-pull-repo"

# Install dependencies
npm install
```

4. Click **Save**.

**Screenshot placeholder:**

```
![Screenshot: Configure node-install-deps Build Step](screenshots/jenkins-node-install-deps-build-step.png)
```

---

## 🚀 Job 3 — node-deploy-app (Deploy application)

1. Jenkins → **New Item** → **Freestyle project** → Name: `node-deploy-app`
2. **Build Triggers** → **Build after other projects are built** → Enter `node-install-deps`
3. **Build Steps** → **Execute shell** → Script:

```bash
# Optionally call previous job (not necessary if you chain builds)
# cd to workspace
cd "/var/lib/jenkins/workspace/node-pull-repo"

# Start or restart the app using PM2
pm2 start app.js --name node-app || pm2 restart node-app

# Save pm2 process list so it survives restarts (optional but recommended)
pm2 save

# If you want PM2 to resurrect on server boot, generate startup script (runs once)
# Do this as the jenkins user. Example for systemd:
# sudo su - jenkins -s /bin/bash -c "pm2 startup systemd -u jenkins --hp /var/lib/jenkins"
# Then follow the printed command (it will ask to run as root once to register the systemd script).
```

4. Click **Save**.

**Screenshot placeholder:**

```
![Screenshot: Configure node-deploy-app Build Step](screenshots/jenkins-node-deploy-app-build-step.png)
```

---

## 🔁 GitHub Webhook (for auto build on push)

1. In your GitHub repo → **Settings** → **Webhooks** → **Add webhook**
2. **Payload URL**: `http://<JENKINS_HOST>/github-webhook/`
3. **Content type**: `application/json`
4. **Which events**: `Just the push event` (or choose `Let me select individual events`)
5. Click **Add webhook**.

**Screenshot placeholder:**

```
![Screenshot: GitHub webhook settings](screenshots/github-webhook.png)
```

---

## ✅ Verify the flow

1. Push code to the configured branch on GitHub.
2. Check the webhook delivery status in GitHub (it should show 200 OK).
3. In Jenkins, watch the build queue and the console output for `node-pull-repo`, `node-install-deps`, and `node-deploy-app`.
4. Check PM2 process list: `pm2 list` (as `jenkins` user) and `pm2 logs node-app` for runtime logs.

---

## 🐞 Troubleshooting

* **npm not found in Jenkins job**: Ensure `npm` is in PATH for `jenkins` user. Verify with `sudo su - jenkins -s /bin/bash -c "which npm && npm -v"`.
* **PM2 permission errors**: Make sure PM2 was installed for `jenkins` user and that `jenkins` owns the pm2 files. Re-install as `jenkins` if needed.
* **App fails to start**: Check `pm2 logs node-app` and `node` version compatibility.
* **Port already in use**: Verify the application port and stop any other process using it.

---

## 📂 Screenshots folder (project structure suggestion)

Add screenshots to your repository in a `screenshots` folder so they are easily viewable in README:

```
repo-root/
  ├─ README.md
  └─ screenshots/
      ├─ jenkins-create-node-pull-repo.png
      ├─ jenkins-node-install-deps-build-step.png
      ├─ jenkins-node-deploy-app-build-step.png
      └─ github-webhook.png
```

> **Tip:** Use the exact filenames used in the image placeholders above so Markdown will render them automatically on GitHub.

---

## 📌 Extras & Recommendations

* Use **blue-green** or **rolling** deployment strategies for zero-downtime deployments.
* Consider using the **Jenkins Pipeline** (Declarative or Scripted) for better reproducibility and versioning of your deployment logic.
* If you need specific Node.js versions, install `nvm` for the `jenkins` user and set the desired Node version in the job shell steps.

---

## ✍️ If you want me to:

* Add real screenshots into these placeholders — upload the images here and I will place them in the README and update image links.
* Convert this into a Jenkins Pipeline `Jenkinsfile` — I can write a Declarative Pipeline that does the same steps.

---

*Prepared for quick copy/paste. Replace any paths, repo URLs, and filenames to match your setup.*
