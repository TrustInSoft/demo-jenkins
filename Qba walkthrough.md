# Jenkins demo on TinyCrypt

## Jenkins installation

NOTE: This is an internal note, so that you can see exactly what I did on my machine to install Jenkins.  
Please rather follow what is in `README.md` and use this only as reference for this first part.

### Download and install

```
$ wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt update 
$ sudo apt install jenkins
```

### Start

```
$ sudo systemctl start jenkins
```

### Check status

```
$ sudo systemctl status jenkins
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2022-12-22 16:15:50 CET; 1min 4s ago
 Main PID: 31292 (java)
    Tasks: 57 (limit: 4915)
```

### Get the admin password

```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
4660f61ba95b4bdd86ccca59a183438f
```

- Connect to `4660f61ba95b4bdd86ccca59a183438f` on a web browser.
- Input the admin password.
- Click on "Install suggested plugins".

### Create the First Admin User

- username: `qba`
- password: `MyLittleJenkins!`
- name: Jakub Zwolakowski
- e-mail: `jakub.zwolakowski@trust-in-soft.com`

### Access

Jenkins URL: `http://localhost:8080/`

## Jenkins configuration

- **NOTE**: This is improved configuration, use this instead of what is in `README`.

### Jenkins Freestyle

Create and configure a **Freestyle Project**:

- Click **Create a Job**
- Click **Freestyle project**
- Section **Source Code Management**
  - Set **Git->Repositories->Repository URL** to `https://github.com/TrustInSoft/demo-jenkins.git`
  - Click **Add Repository**
- Section **Build Triggers**
  - Choose **Build periodically**
  - Set **Schedule** to `H * * * *`
- Section **Build Environment**
  - Choose **Delete workspace before build starts**
  - Choose **Inject environment variables to the build process**
    - Set **Properties Content** to `TIS_VERSION=1.44`
- Section **Build Steps**
  - Insert all the **Build Steps** and **Post-Build Actions**
    - Add step **Execute shell** (1)
      ```bash
      /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
      source ~/.tis.conf
      trustinsoft/run_all.sh -n 1
      ```
    - Add step **Execute shell** (2)
      ```bash
      /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
      source ~/.tis.conf
      tis-report _results
      ```
    - Add step **Execute shell** (3)
      ```bash
      /home/tis/${TIS_VERSION}/bin/tis-setenv.sh
      source ~/.tis.conf
      tis-misra lib
      ```
- Section **Post-build Actions**
  - Add step **Archive the artifacts**
    - Set **Files to archive** to  
      `tis_report.html, _results/**.json, _results/**.csv, tis_misra_report/**`

### Jenkins Pipeline

Same as in **README** (note, that I've fixed and updated the `Jenkinsfile`).

### Jenkins global setup

Global Jenkins configuration:

- Section **Shell**
  - Set **Shell executable** to `/bin/bash`

## TODO

- What about the parallelization variable for the pipeline?
- What about the `${TIS_VERSION}` for the pipeline?
