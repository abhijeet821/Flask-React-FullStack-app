# Flask + React Full-Stack App Deployment with GitHub Actions

## Overview
This project is a fully working Flask (Python) + React (JavaScript) full-stack application with CRUD operations, deployed locally using **Nginx** and **Gunicorn** for production serving.

The deployment process is automated using **GitHub Actions** with a **self-hosted runner** running directly on the deployment machine.

---

## 1. Local Deployment Setup
- **Backend:** Flask app with `Flask`, `Flask-SQLAlchemy`, and `flask-cors`.
- **Frontend:** React app built with Vite.
- **Nginx Reverse Proxy**:
  - Serves frontend build files from `/frontend/dist`.
  - Forwards API requests to Gunicorn running the Flask backend.
- **Gunicorn** managed via `systemd` service for automatic restarts and background running.

---

## 2. CI/CD Pipeline (Implemented)
We used **GitHub Actions** with a self-hosted runner installed on the local deployment machine.

**Trigger:** Push to the `deploy` branch.

**Workflow Steps:**
1. **Linting**  
   - Backend: `flake8` for Python code quality.  
   - Frontend: `eslint` for JavaScript code quality.
2. **Pull Latest Code** from the `deploy` branch.
3. **Install Dependencies** for backend (`pip install -r requirements.txt`) and frontend (`npm install`).
4. **Build Frontend** with `npm run build`.
5. **Restart Backend** via `systemctl restart gunicorn`.
6. **Reload Nginx** to serve the new frontend build.

Since the runner is installed on the deployment machine, no SSH is required.

---

## 3. How It Would Work with EC2 SSH Deployment (If Implemented)
If AWS EC2 was used instead of a local/self-hosted runner:
- GitHub Actions would SSH into the EC2 instance using a private key stored in GitHub Secrets.
- Steps after SSH:
  - Pull latest code from repo.
  - Install dependencies.
  - Build frontend.
  - Restart Gunicorn and reload Nginx.
- This would enable cloud hosting but require SSH key management and EC2 setup.

---

## 4. How It Would Work with Domain + SSL (If Implemented)
If a custom domain was registered:
- The domain’s **A record** would point to the server's public IP.
- Nginx would be configured to respond to that domain.
- HTTPS would be enabled using **Certbot** and Let’s Encrypt.
- SSL certificates would be auto-renewed every 90 days for security.

---

## 5. Problems Faced & Solutions

### 1. Self-Hosted Runner Network Failures
While setting up the self-hosted runner, dependency installation failed with network errors.  
I examined the error logs and saw it was failing to connect to `npm` and `pypi`. This meant the runner environment had no internet access. I paused the workflow and tested connectivity manually before retrying the installation.

---

### 2. GitHub Actions Stuck on `sudo` Commands
When the workflow reached the step to restart Gunicorn or reload Nginx, it got stuck indefinitely.  
I suspected it was waiting for a password prompt. On checking the runner logs, I confirmed that `sudo` was asking for a password. This meant the workflow could never complete without modifying permissions.

---

### 3. Gunicorn Restart Hanging
Even after allowing `sudo` commands, the restart step for Gunicorn would sometimes hang for too long.  
I checked the system logs (`journalctl`) and found that Gunicorn workers were not shutting down quickly, causing the restart to wait indefinitely. We then thought of handling old processes manually before starting a fresh instance, so I killed any past instances of Gunicorn and the started a fresh instance, this solved the problem.


---

Overall, these issues forced us to repeatedly pause the automation, inspect logs, and manually verify each service’s state before running the workflow again. This helped refine the final deployment process so that it now runs reliably.


**Summary:**  
The implemented setup achieves automated deployment for a Flask + React app without relying on manual steps. While the current system uses a local self-hosted runner, the same CI/CD pipeline can be extended to work with EC2 SSH deployments, custom domains, and SSL certificates.
