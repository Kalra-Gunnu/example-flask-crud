# Flask CRUD App CI/CD with Jenkins on EC2  

This guide walks through setting up Jenkins on an EC2 instance, configuring a CI/CD pipeline for a Flask CRUD application, and deploying it with Gunicorn and Systemd.  

---

## 1. Launch EC2 Instance  

<img src="screenshots/1.png" width="700">  

---

## 2. SSH into EC2 & Update Packages  

```bash
sudo apt update  
sudo apt upgrade -y
```
Then install JDK 17:

```bash
sudo apt install openjdk-17-jre -y
```
<img src="screenshots/2.png" width="700">
3. Verify Java Installation

```bash
java --version  
```
<img src="screenshots/3.png" width="700">
4. Install Jenkins
Add Jenkins repo & install in a single command:

```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null && \
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null && \
sudo apt-get update && \
sudo apt-get install -y jenkins
```
<img src="screenshots/4.png" width="700">
5. Enable & Start Jenkins

```bash
sudo systemctl enable jenkins  
sudo systemctl start jenkins
``` 
Check Jenkins version:

```bash
jenkins --version
```
<img src="screenshots/5.png" width="700">
6. Access Jenkins
Visit http://<EC2_PUBLIC_IP>:8080

To get the initial password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```  
<img src="screenshots/6.png" width="700"> <img src="screenshots/7.png" width="700">
7. Setup Jenkins
Enter the admin password

Click Install suggested plugins

Create the first admin user

Keep default configs & proceed

<img src="screenshots/8.png" width="700"> <img src="screenshots/9.png" width="700"> <img src="screenshots/10.png" width="700">
8. Install Additional Plugins
Go to Manage Jenkins → Available Plugins and install required plugins.

<img src="screenshots/11.png" width="700">
9. Fork Flask App Repository
Fork example-flask-crud

<img src="screenshots/12.png" width="700">
10. Clone Forked Repo on Jenkins Server

```bash
cd ~  
git clone <your_forked_repo_url>  
ls
```
<img src="screenshots/13.png" width="700">
11. Allocate & Associate Elastic IP
<img src="screenshots/14.png" width="700">
12. Create Jenkinsfile
Go into the app directory:

```bash
cd <app_name>  
sudo nano Jenkinsfile
```
<img src="screenshots/15.png" width="700">
13. Add Tests
Create tests/test_app.py with a sample test.

<img src="screenshots/16.png" width="700">
14. Commit & Push Changes

```bash
git add .  
git commit -m "Added Jenkinsfile & tests"  
git push origin master
```
<img src="screenshots/17.png" width="700">
15. Setup SSH Key for GitHub
Generate SSH key & add it to GitHub → Settings → SSH Keys

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"  
cat ~/.ssh/id_ed25519.pub
``` 
Then test connection:

```bash
ssh -T git@github.com
```
Update repo remote:

```bash
git remote set-url origin git@github.com:<username>/<repo_name>.git
```
<img src="screenshots/18.png" width="700">
16. Create Jenkins Pipeline
Go to Dashboard → New Item

Name: Flask-CRUD-Pipeline

Type: Pipeline

Pipeline from SCM → Git

Repo URL: https://github.com/<your_user>/example-flask-crud.git

Branch: */master

Script Path: Jenkinsfile

<img src="screenshots/19.png" width="700">
17. Enable Poll SCM
Go to Pipeline → Configure → Build Triggers

Check Poll SCM

Schedule:  H/5 * * * *
<img src="screenshots/20.png" width="700">
18. Install Python venv

```bash
sudo apt install python3-venv -y
```
<img src="screenshots/21.png" width="700">
19. First Build
Run the pipeline manually.

✅ Build successful but Flask app didn’t stay running because nohup flask run & was killed after job completion.

<img src="screenshots/22.png" width="700">
20. Fix Deployment Stage
Updated Jenkinsfile to detach process properly. SCM Poll triggered an auto build.

<img src="screenshots/23.png" width="700">
21. Use Gunicorn & Systemd
Installed Gunicorn and created a Systemd service:

```ini
[Unit]
Description=Flask CRUD App
After=network.target

[Service]
User=jenkins
Group=jenkins
WorkingDirectory=/var/lib/jenkins/workspace/Flask-CRUD-Pipeline
ExecStart=/var/lib/jenkins/workspace/Flask-CRUD-Pipeline/venv/bin/gunicorn --workers 2 --bind 0.0.0.0:5000 crudapp:app
Restart=always
Environment="PATH=/var/lib/jenkins/workspace/Flask-CRUD-Pipeline/venv/bin"

[Install]
WantedBy=multi-user.target
```
Reload & restart:

```bash
sudo systemctl daemon-reexec  
sudo systemctl restart flaskcrud  
sudo systemctl status flaskcrud
```
<img src="screenshots/24.png" width="700">
22. Fix Sudo in Jenkins
Edit sudoers file:

```bash
sudo visudo  
```
Add:

```bash
Defaults:jenkins !requiretty
jenkins ALL=(ALL) NOPASSWD: /bin/systemctl restart flaskcrud, /bin/systemctl status flaskcrud, /bin/systemctl enable flaskcrud
```
Then rerun the pipeline manually.

<img src="screenshots/25.png" width="700">
23. Final Deployment
Ran the pipeline again → ✅ Flask app successfully running at http://<Elastic_IP>:5000

<img src="screenshots/26.png" width="700">
🎉 Final Outcome
✅ Jenkins pipeline auto-builds on every push
✅ Flask app deployed via Gunicorn + Systemd
✅ SCM Poll triggers auto-deploy

