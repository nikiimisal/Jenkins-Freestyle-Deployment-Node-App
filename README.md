# Node.js App Deployment using Jenkins Freestyle Projects

This guide explains step-by-step how to set up Jenkins and deploy a simple Node.js app. It is beginner-friendly and divided into four Jenkins jobs.

---

## 🧩 What You Will Do

We’ll create **four Jenkins jobs**:

0. **set-up-env** → installs Node.js, npm, and PM2.
1. **node-pull-repo** → pulls your code from GitHub.
2. **node-install-deps** → installs dependencies.
3. **node-deploy-app** → runs or restarts the app using PM2.

---

## ⚙️ Job 0 — set-up-env (Prepare the environment)

### Steps:

1. Go to Jenkins → **New Item** → choose **Freestyle Project** → name it `set-up-env`.
2. Add a **Build Step → Execute Shell** and paste:

```bash
sudo apt update -y
sudo apt install nodejs npm -y
sudo npm install -g pm2

# Optional: run under Jenkins user
sudo su - jenkins -s /bin/bash -c "npm install -g pm2"
sudo su - jenkins -s /bin/bash -c "node -v && npm -v && pm2 -v"
```

3. Under **Post-build Actions**, select **Build other projects** → Enter `node-pull-repo`.
4. Click **Save**.

This job prepares the Jenkins environment for Node.js apps.

---

## 🧾 Job 1 — node-pull-repo (Pull Repository)

1. Create a new Freestyle project named `node-pull-repo`.
2. Under **Source Code Management**, select **Git**.

   * Repository URL: `https://github.com/iamtruptimane/node-js-app-CICD.git`
   * Branch: `main`
3. (Optional) Under **Build Triggers**, check **GitHub hook trigger for GITScm polling**.
4. Leave **Build Steps** empty — this job just pulls code.
5. Under **Post-build Actions**, select **Build other projects** → Enter `install-dependency`.
6. Click **Save**.

---

## 🧱 Job 2 — node-install-deps (Install Dependencies)

1. Create a new Freestyle project named `node-install-deps`.
2. (Optional) Under **Build Triggers**, select **Build after other projects are built** → enter `node-pull-repo`.
3. Add a **Build Step → Execute Shell**:

```bash
cd /var/lib/jenkins/workspace/node-pull-repo
sudo npm install

```

4. Under **Post-build Actions**, select **Build other projects** → Enter `node-deploy-app`.
5. Click **Save**.

---

## 🚀 Job 3 — node-deploy-app (Deploy the App)

1. Create a new Freestyle project named `node-deploy-app`.
2. Under **Build Triggers**, select **Build after other projects are built** → enter `node-install-deps`.
3. Add a **Build Step → Execute Shell**:

```bash
cd /var/lib/jenkins/workspace/node-pull-repo
pm2 start app.js --name node-app || pm2 restart node-app
pm2 save
```

4. Click **Save**.

---

## 🔁 Setup Job Connections

To make jobs run automatically in sequence:

* `set-up-env` → Run this **manually** the first time only.
* `node-pull-repo` → Triggered automatically by GitHub webhook.
* `node-install-deps` → Triggered **after ****`node-pull-repo`** completes.
* `node-deploy-app` → Triggered **after ****`node-install-deps`** completes.

This ensures a smooth chain:
**Git push → Jenkins pulls repo → Installs deps → Deploys app.**

---

## 🌐 GitHub Webhook Setup ( Optional )

1. Go to your GitHub repo → **Settings → Webhooks → Add webhook**.
2. Payload URL: `http://<your-jenkins-server>/github-webhook/`
3. Content type: `application/json`
4. Trigger: **Just the push event**
5. Click **Add webhook**.

---

## ✅ Test the Full Flow

1. Run `set-up-env` once manually.
2. Push code to GitHub → `node-pull-repo` runs automatically.
3. Jenkins will then run `node-install-deps` → `node-deploy-app`.
4. Check running apps:

```bash
sudo su - jenkins -s /bin/bash -c "pm2 list"
```

5. View logs:

```bash
sudo su - jenkins -s /bin/bash -c "pm2 logs node-app"
```

---

## 🧠 Common Issues

| Problem               | Cause              | Fix                                             |
| --------------------- | ------------------ | ----------------------------------------------- |
| npm not found         | Jenkins PATH issue | Install npm for `jenkins` user or use full path |
| PM2 permission denied | Installed as root  | Reinstall PM2 under `jenkins` user              |
| App not starting      | Wrong file or port | Verify `app.js` and port availability           |

---

## 🏁 Summary

After completing all steps:

* Jenkins pulls your GitHub repo automatically.
* Installs dependencies automatically.
* Starts or restarts your Node.js app via PM2.

✅ You now have a complete **CI/CD pipeline using Jenkins Freestyle Projects** for Node.js!
