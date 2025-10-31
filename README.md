# Node.js App Deployment using Jenkins Freestyle Projects

This guide will help you step-by-step to set up Jenkins and deploy a simple Node.js app. It is written in a beginner-friendly way so anyone can follow it easily.

---

## 🧩 What You Will Do

We’ll create **four Jenkins jobs** to manage everything from setup to deployment:

1. **set-up-env** → installs Node.js, npm, and PM2.
2. **node-pull-repo** → pulls your code from GitHub.
3. **node-install-deps** → installs dependencies.
4. **node-deploy-app** → runs or restarts the app using PM2.

---

## ⚙️ Job 0 — set-up-env (Prepare the environment)

This job makes sure your Jenkins server has Node.js, npm, and PM2 installed so the `jenkins` user can run them directly.

### Steps:

1. Go to Jenkins → **New Item** → choose **Freestyle Project** → name it `set-up-env`.
2. In **Build Steps**, select **Execute Shell** and paste these commands:

```bash
# Update system packages
sudo apt update -y

# Install Node.js (use nodejs or node based on your system)
sudo apt install -y nodejs

# Install npm
sudo apt install -y npm

# Install PM2 globally for Jenkins user
# This makes PM2 available everywhere, so you can use it in any directory.
sudo su - jenkins -s /bin/bash -c "npm install -g pm2"

# Check versions to make sure everything worked
sudo su - jenkins -s /bin/bash -c "node -v && npm -v && pm2 -v"
```

### Why we use `-g` while installing PM2:

* The `-g` flag installs PM2 globally, so you can run the `pm2` command from **any directory**.
* Without `-g`, PM2 would only work inside the specific project folder.

> If you skip `-g`, you can still run PM2 locally using `npx pm2 start app.js`, but global installation is easier for Jenkins.

3. Click **Save** and **Build Now**.

After this job succeeds, Jenkins will be ready to build and deploy Node.js apps.

**Screenshot placeholder:**

```
![Screenshot: set-up-env job](screenshots/set-up-env-job.png)
```

---

## 🧾 Job 1 — node-pull-repo (Pull Repository)

1. Create a new Freestyle project named `node-pull-repo`.
2. In **Source Code Management**, select **Git**.

   * Repository URL: `https://github.com/iamtruptimane/node-js-app-CICD.git`
   * Branch: `main`
3. In **Build Triggers**, check **GitHub hook trigger for GITScm polling**.
4. Leave **Build Steps** empty — this job only pulls the code.
5. Click **Save**.

**Screenshot placeholder:**

```
![Screenshot: node-pull-repo job](screenshots/node-pull-repo.png)
```

---

## 🧱 Job 2 — node-install-deps (Install Dependencies)

1. Create a new Freestyle project named `node-install-deps`.
2. In **Build Triggers**, select **Build after other projects are built** → enter `node-pull-repo`.
3. In **Build Steps → Execute Shell**, paste:

```bash
cd /var/lib/jenkins/workspace/node-pull-repo
npm install
```

4. Click **Save**.

**Screenshot placeholder:**

```
![Screenshot: node-install-deps job](screenshots/node-install-deps.png)
```

---

## 🚀 Job 3 — node-deploy-app (Deploy the App)

1. Create a new Freestyle project named `node-deploy-app`.
2. In **Build Triggers**, choose **Build after other projects are built** → enter `node-install-deps`.
3. In **Build Steps → Execute Shell**, paste:

```bash
cd /var/lib/jenkins/workspace/node-pull-repo
pm2 start app.js --name node-app || pm2 restart node-app
pm2 save
```

4. Click **Save**.

**Screenshot placeholder:**

```
![Screenshot: node-deploy-app job](screenshots/node-deploy-app.png)
```

---

## 🔁 Setup GitHub Webhook (Auto Build on Push)

1. Go to your GitHub repo → **Settings** → **Webhooks** → **Add webhook**.
2. In **Payload URL**, enter:

   ```
   http://<your-jenkins-server>/github-webhook/
   ```
3. Set **Content type** to `application/json`.
4. Select **Just the push event**.
5. Click **Add webhook**.

**Screenshot placeholder:**

```
![Screenshot: GitHub webhook](screenshots/github-webhook.png)
```

---

## ✅ Test the Full Flow

1. Run the first job (`set-up-env`) once to prepare the system.
2. Push code to GitHub → it should trigger `node-pull-repo` automatically.
3. After that, Jenkins will run `node-install-deps` → `node-deploy-app`.
4. To check the running app:

   ```bash
   sudo su - jenkins -s /bin/bash -c "pm2 list"
   ```
5. To see logs:

   ```bash
   sudo su - jenkins -s /bin/bash -c "pm2 logs node-app"
   ```

---

## 🧠 Common Problems and Fixes

| Problem               | Cause                   | Fix                                             |
| --------------------- | ----------------------- | ----------------------------------------------- |
| npm not found         | Jenkins PATH issue      | Install npm for `jenkins` user or use full path |
| PM2 permission denied | Installed as root       | Reinstall PM2 as `jenkins` user                 |
| App not starting      | Wrong file name or port | Verify `app.js` path and free the port          |

---

## 📂 Recommended Folder for Screenshots

Keep screenshots in a folder named `screenshots/` inside your repo, like this:

```
project-root/
 ├─ README.md
 └─ screenshots/
     ├─ set-up-env-job.png
     ├─ node-pull-repo.png
     ├─ node-install-deps.png
     ├─ node-deploy-app.png
     └─ github-webhook.png
```

---

## 🏁 Summary

After completing all steps:

* Jenkins can pull your code from GitHub automatically.
* Dependencies will install automatically.
* Your Node.js app will start or restart automatically using PM2.

You now have a complete **CI/CD setup for a Node.js app** using **Jenkins Freestyle Projects**!
