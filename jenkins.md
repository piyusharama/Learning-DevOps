Here is a 6-lecture course plan designed for noob students, focusing on Jenkins CI/CD with step-by-step guides, culminating in a simple "hello world" HTML application deployment to an EC2 instance without initially involving Docker or Kubernetes.

***

### Jenkins CI/CD for Beginners: A 6-Lecture Course

**Course Goal:** By the end of this course, students will understand the fundamentals of Continuous Integration and Continuous Delivery (CI/CD) using Jenkins and be able to deploy a simple web application to an AWS EC2 instance.

---

### **Lecture 1: Introduction to Jenkins and CI/CD Fundamentals (3 hours)**

**Learning Objectives:**
*   Understand what Jenkins is and its role in DevOps.
*   Grasp the core concepts of Continuous Integration (CI) and Continuous Delivery (CD).
*   Learn how to install Jenkins on an AWS EC2 instance.

**Content & Step-by-Step Guide:**

1.  **What is Jenkins?**
    *   **Definition:** Jenkins is an open-source automation server. It helps automate various parts of software development, including building, testing, and deploying applications.
    *   **Role in DevOps:** Jenkins acts as a pipeline orchestrator, managing continuous integration and continuous delivery pipelines.
    *   **Why Jenkins?** It's a highly in-demand DevOps skill, automates repetitive tasks, integrates with a wide range of tools like Git and Maven, boosts team collaboration, is scalable, and offers career growth opportunities.

2.  **Introduction to CI/CD:**
    *   **Traditional Software Development Lifecycle (SDLC) Issues:** Discuss problems like time-consuming processes, unproductive changes, lack of transparency, and management bottlenecks.
    *   **Continuous Integration (CI):**
        *   **Concept:** Developers frequently add their code changes to a shared repository (like GitHub). Automated tools then build and test these changes.
        *   **Benefits:** Helps find and address bugs quickly, frees developers from manual tasks, and allows more frequent updates.
    *   **Continuous Delivery (CD):**
        *   **Concept:** An automated process to get software from version control all the way to users. It ensures code is always ready for deployment, though a manual approval might be required before deploying to production.
    *   **Continuous Deployment:** An extension of CD, where code is automatically deployed to production without any manual intervention.
    *   **Core Difference:** The key difference between Continuous Delivery and Continuous Deployment lies in the manual approval step for production deployment.
    *   **Jenkins Workflow Overview:** Code is committed to a Version Control System (VCS), Jenkins triggers a build (often via a webhook), runs tests (including static code analysis), uploads artifacts to a repository, and then deploys to various environments (Dev, QA, Production) with continuous feedback and monitoring at each stage.

3.  **Jenkins Prerequisites:**
    *   **Java:** Jenkins is developed in Java, so **Java Runtime Environment (JRE) version 8 or 11 (or later supported versions like 17, 21)** is required on the system where Jenkins will be installed.
    *   **Hardware:** A minimum of **256MB RAM** and **1GB drive space** are needed for a small Jenkins installation, though **4GB RAM** and **50GB drive space** are recommended for better performance.

4.  **Installing Jenkins on AWS EC2 (Step-by-Step):**

    *   **Step 1: Launch an EC2 Instance on AWS**
        *   Log in to your AWS Management Console.
        *   Go to **EC2** service.
        *   Click **"Launch Instances"**.
        *   **Name:** `Jenkins-Master`.
        *   **Application and OS Images (AMI):** Select **Ubuntu Server** (e.g., 22.04 LTS, x86_64).
        *   **Instance Type:** Select `t2.micro` (free tier eligible).
        *   **Key pair (login):** Create a **new key pair** (e.g., `jenkins-key`) and download the `.pem` file. Keep it secure.
        *   **Network settings:**
            *   **Security group:** Create a **new security group**.
            *   **Inbound security group rules:**
                *   Add rule: **SSH** (Port 22), Source: `My IP` (for your local machine to connect).
                *   Add rule: **Custom TCP** (Port 8080), Source: `Anywhere` (for Jenkins UI access from anywhere for this demo, or `My IP` for stricter security).
        *   **Configure storage:** Default 8 GiB is usually sufficient for initial setup; you can increase it later if needed.
        *   Click **"Launch Instance"**. Wait for the instance state to become `Running`.

    *   **Step 2: Connect to the EC2 Instance**
        *   Open your terminal (Linux/macOS) or use PuTTY (Windows).
        *   Navigate to the directory where your `.pem` key file is downloaded.
        *   Set correct permissions for your key: `chmod 400 jenkins-key.pem` (Linux/macOS).
        *   Connect to your instance:
            `ssh -i jenkins-key.pem ubuntu@<your-ec2-public-ip-address>`
            (Replace `<your-ec2-public-ip-address>` with your instance's public IP).
            Confirm connection by typing `yes` when prompted.

    *   **Step 3: Install Java (OpenJDK 17) on EC2**
        *   Update package list: `sudo apt update`.
        *   Install OpenJDK 17: `sudo apt install -y openjdk-17-jre-headless`.
        *   Verify Java installation: `java --version`.

    *   **Step 4: Install Jenkins on EC2**
        *   Add Jenkins GPG key:
            `sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key`.
        *   Add Jenkins repository to sources list:
            `echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null`.
        *   Update package list again: `sudo apt update`.
        *   Install Jenkins: `sudo apt install -y jenkins`.

    *   **Step 5: Start and Enable Jenkins Service**
        *   Start Jenkins: `sudo systemctl start jenkins`.
        *   Enable Jenkins to start on boot: `sudo systemctl enable jenkins`.
        *   Check status: `sudo systemctl status jenkins` (should show `active (running)`).

    *   **Step 6: Access Jenkins Web UI and Initial Setup**
        *   Open your web browser and go to `http://<your-ec2-public-ip-address>:8080`.
        *   **Unlock Jenkins:** Copy the initial admin password from the EC2 instance:
            `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`.
            Paste the password into the Jenkins UI and click `Continue`.
        *   **Install Plugins:** Select **"Install suggested plugins"**. Jenkins will download and install a set of commonly used plugins.
        *   **Create Admin User:** Create a new admin user (e.g., username `admin`, password `admin`).
        *   **Jenkins Ready:** Click `Save and Finish`, then `Start using Jenkins`.

---

### **Lecture 2: Jenkins Jobs - Freestyle Project (3 hours)**

**Learning Objectives:**
*   Navigate the Jenkins user interface.
*   Understand Jenkins jobs (projects) and their types.
*   Create and configure a basic Freestyle project to clone code and run shell commands.
*   Explore the Jenkins workspace.

**Content & Step-by-Step Guide:**

1.  **Jenkins UI Tour:**
    *   **Dashboard:** Overview of jobs, build queue, build executor status.
    *   **New Item:** To create new jobs/projects.
    *   **Build History:** Shows past builds and their status.
    *   **Manage Jenkins:** Administration, plugins, security, system configuration.

2.  **Jenkins Jobs (Projects):**
    *   **Concept:** The fundamental unit of work in Jenkins. A job defines a sequence of steps that Jenkins will execute, such as building software, running tests, or deploying applications.
    *   **Types of Projects (Overview):** Freestyle project, Pipeline, Multibranch Pipeline, Folder, etc..
    *   **Focus:** **Freestyle Project** – highly flexible, allows for arbitrary build steps and actions.

3.  **Creating a Simple "Hello World" Freestyle Project (Step-by-Step):**

    *   **Step 1: Create a New Item**
        *   From the Jenkins Dashboard, click **"New Item"** (or "Create a job").
        *   **Item name:** `HelloWorldFreestyle`.
        *   Select **"Freestyle project"**.
        *   Click **"OK"**.

    *   **Step 2: Configure General Settings**
        *   **Description:** `A simple job to display "Hello World" and clone a GitHub repository.`
        *   (Optional) **Discard old builds:** Check this to limit the number of builds and save disk space. For noobs, keep it unchecked initially.

    *   **Step 3: Configure Source Code Management (SCM) - GitHub**
        *   **Prerequisite:** Create a **public GitHub repository** (e.g., `https://github.com/your-username/hello-world-html`) containing a simple `index.html` file.
        *   Under **Source Code Management**, select **"Git"**.
        *   **Repository URL:** Paste your GitHub repository's HTTPS URL (e.g., `https://github.com/your-username/hello-world-html.git`).
        *   **Credentials:** Leave `None` as it's a public repository.
        *   **Branches to build:** Keep `*/master` or change to `*/main` if your default branch is `main`.

    *   **Step 4: Configure Build Triggers (Manual for now)**
        *   For this lecture, leave all build triggers **unchecked**. Builds will be triggered manually using "Build Now".

    *   **Step 5: Configure Build Steps - "Execute Shell"**
        *   Under **Build Steps**, click **"Add build step"**.
        *   Select **"Execute shell"**.
        *   **Command:** Enter the following commands:
            ```bash
            echo "Hello, Jenkins World!"
            echo "Cloning repository to workspace..."
            ls -lah  # List files after cloning
            mkdir -p my_app_output
            echo "A new directory 'my_app_output' was created."
            ```
            *(Note: When you configure Git SCM, Jenkins automatically clones the repository into the job's workspace before executing build steps, so `git clone` command might not be explicitly needed here if the job is set up correctly for SCM.)*
        *   Click **"Add build step"** again.
        *   Select **"Execute shell"**.
        *   **Command:** `ls -l /var/lib/jenkins/workspace/HelloWorldFreestyle` (to see the cloned files).

    *   **Step 6: Save and Build the Job**
        *   Click **"Apply"**, then **"Save"**.
        *   On the job's page, click **"Build Now"**.
        *   Observe the **"Build History"** section, a new build will appear.
        *   Click on the build number (e.g., `#1`), then select **"Console Output"**.
        *   Review the output: You should see "Hello, Jenkins World!", confirmation of cloning, and the files listed from your GitHub repository within the Jenkins workspace.

4.  **Jenkins Workspace:**
    *   **Concept:** A temporary directory on the Jenkins server (or agent) where the job's source code is checked out, and build artifacts are created.
    *   **Location:** For a job named `HelloWorldFreestyle`, the workspace is typically at `/var/lib/jenkins/workspace/HelloWorldFreestyle` on your Jenkins Master.
    *   **Demonstration:** Show how to SSH into the Jenkins Master and navigate to the workspace directory to verify the cloned files (`cd /var/lib/jenkins/workspace/HelloWorldFreestyle`, then `ls -lah`).

5.  **Troubleshooting (Common "noob" issues):**
    *   **Permission Denied:** Explain that shell commands might fail due to insufficient permissions. Briefly mention `sudo` or changing file permissions (e.g., `chmod`) as potential solutions. For simplicity, advise using `sudo` for system-level commands within Jenkins shell steps for now, but highlight it's not a best practice for production (will be addressed in later lectures).

---

### **Lecture 3: Introduction to Jenkins Pipelines - Declarative Pipeline (3 hours)**

**Learning Objectives:**
*   Understand the "Pipeline as Code" concept and its benefits.
*   Differentiate between Declarative and Scripted Pipelines, focusing on Declarative.
*   Write a basic Declarative Pipeline (`Jenkinsfile`) with multiple stages.
*   Configure Jenkins to read `Jenkinsfile` from Source Code Management (SCM).
*   Set up basic build triggers like GitHub Webhooks and Poll SCM.

**Content & Step-by-Step Guide:**

1.  **What is Jenkins Pipeline?**
    *   **Concept:** A suite of plugins that supports implementing and integrating Continuous Delivery pipelines into Jenkins. It allows defining your CI/CD flow "as code" within a text file called a `Jenkinsfile`.
    *   **Benefits:** **Version Control** (Jenkinsfile is in SCM), **Easy to Migrate and Maintain**, supports **Parallel Task Execution**, allows **Complex Conditional Logic**, and enables **Dynamic Resource Allocation**.

2.  **Declarative vs. Scripted Pipeline:**
    *   **Declarative Pipeline (Focus):** Easier to learn for beginners. It uses a **predefined structure and model** to create consistent, concise pipelines without extensive Groovy knowledge.
    *   **Scripted Pipeline:** More flexible, allows coding Pipelines using a Groovy DSL, but requires deeper Groovy knowledge. (Briefly mention, but do not teach in detail).

3.  **Basic Declarative Pipeline Syntax:**
    *   Every Declarative Pipeline starts with a `pipeline` block.
    *   `agent`: Defines where the Pipeline or a subset of it will execute.
        *   `agent any`: Executes on any available agent.
        *   `agent none`: No agent specified at the top level, agents must be defined per stage.
        *   `agent { node { label 'my-agent' } }`: Specifies a particular agent by its label.
    *   `stages`: Contains all the `stage` blocks.
    *   `stage`: A specific named "Stage" of the Pipeline, representing a logical step (e.g., "Build", "Test", "Deploy").
    *   `steps`: Contains the actual actions or commands to be executed within a stage (e.g., `sh 'command'`).

4.  **Creating a Simple "Hello World" Declarative Pipeline (Step-by-Step):**

    *   **Step 1: Create a New Pipeline Item**
        *   From Jenkins Dashboard, click **"New Item"**.
        *   **Item name:** `HelloWorldPipeline`.
        *   Select **"Pipeline"**.
        *   Click **"OK"**.

    *   **Step 2: Configure Pipeline Script (Initial Manual Entry)**
        *   In the job configuration, scroll down to the **"Pipeline"** section.
        *   For **"Definition"**, select **"Pipeline script"**.
        *   Copy and paste the following basic `Jenkinsfile` (similar to source):
            ```groovy
            pipeline {
                agent any // Or agent { label 'your_agent_label' } if you have one configured
                stages {
                    stage('Hello') {
                        steps {
                            echo 'Hello, Jenkins Pipeline!'
                        }
                    }
                    stage('Create Folder') {
                        steps {
                            sh 'mkdir -p pipeline-output'
                            echo 'Created folder: pipeline-output'
                        }
                    }
                    stage('Goodbye') {
                        steps {
                            echo 'Pipeline finished successfully.'
                        }
                    }
                }
            }
            ```
        *   Click **"Save"**.

    *   **Step 3: Run the Pipeline and Observe Stages**
        *   On the job's page, click **"Build Now"**.
        *   Observe the **"Stage View"** or **"Build History"**. You'll see the pipeline progress through its defined stages.
        *   Click on the build number and then **"Console Output"** to see the detailed execution logs for each step within stages.

5.  **Installing Pipeline Stage View Plugin:**
    *   **Benefit:** Provides a visual representation of your pipeline stages, making it easier to understand the flow and identify bottlenecks.
    *   **Step-by-Step:**
        *   Go to **"Manage Jenkins"** -> **"Plugins"** -> **"Available plugins"**.
        *   Search for **"Pipeline Stage View"**.
        *   Select it and click **"Install without restart"** (or "Download now and install after restart" if prompted).
        *   After installation, you might need to **restart Jenkins** for it to take full effect (`http://<your-jenkins-ip>:8080/restart`).
        *   After restart, re-run your `HelloWorldPipeline` and observe the enhanced visual Stage View on the job's page.

6.  **`Jenkinsfile` from Source Code Management (SCM):**
    *   **Concept:** Storing your `Jenkinsfile` directly in your project's version control repository (e.g., GitHub) alongside your application code. This ensures that your pipeline definition is version-controlled and changes with your code.
    *   **Step-by-Step:**
        *   **Step 6.1: Create/Update `Jenkinsfile` in GitHub:**
            *   In your `hello-world-html` GitHub repository (from Lecture 2), create a new file named `Jenkinsfile` in the **root directory**.
            *   Copy the `HelloWorldPipeline` script (from Step 2 of this lecture) into this `Jenkinsfile` in GitHub.
            *   Commit the change.
        *   **Step 6.2: Configure Jenkins Pipeline Job:**
            *   Go to `HelloWorldPipeline` job in Jenkins, click **"Configure"**.
            *   Under **"Pipeline"** section, change **"Definition"** from "Pipeline script" to **"Pipeline script from SCM"**.
            *   **SCM:** Select **"Git"**.
            *   **Repository URL:** Paste your `hello-world-html` GitHub repository's HTTPS URL.
            *   **Credentials:** `None`.
            *   **Branches to build:** `*/main` (or `*/master`).
            *   **Script Path:** Enter `Jenkinsfile` (as it's in the root of your repo).
            *   Click **"Save"**.
        *   **Step 6.3: Build and Verify:**
            *   Click **"Build Now"**. Jenkins will now fetch the `Jenkinsfile` directly from your GitHub repository.
            *   Observe the console output to confirm the `Jenkinsfile` was obtained from SCM.

7.  **Build Triggers for Pipelines:**
    *   **GitHub Webhook Trigger:**
        *   **Concept:** Automatically triggers a Jenkins build whenever a change (e.g., a `git push`) is made in the linked GitHub repository. This is the most common and efficient trigger.
        *   **Step-by-Step:**
            *   **7.1.1: Enable Jenkins Webhook:** Go to **"Manage Jenkins"** -> **"Configure System"**. Find the **"GitHub"** section (or similar global settings for SCM integration). Ensure `GitHub hook trigger for GITScm polling` is enabled (often under the `GitHub plugin` configuration).
            *   **7.1.2: Configure GitHub Repository Webhook:**
                *   Go to your `hello-world-html` repository on GitHub.
                *   Click **"Settings"** -> **"Webhooks"**.
                *   Click **"Add webhook"**.
                *   **Payload URL:** `http://<your-jenkins-ip>:8080/github-webhook/`. (Ensure `http` is used if you don't have SSL/HTTPS configured on Jenkins)
                *   **Content type:** Select `application/json`.
                *   **SSL verification:** Select `Disable` for this demo (as we don't have SSL on Jenkins).
                *   **Which events would you like to trigger this webhook?** Select **"Just the push event."** (or "Send me everything" for demo purposes).
                *   Click **"Add webhook"**.
            *   **7.1.3: Enable Trigger in Jenkins Job:**
                *   Go to your `HelloWorldPipeline` job in Jenkins, click **"Configure"**.
                *   Under **"Build Triggers"**, check **"GitHub hook trigger for GITScm polling"**.
                *   Click **"Save"**.
            *   **7.1.4: Test Automation:** Make a small change in your `index.html` file in GitHub directly or via `git push`. Observe that Jenkins automatically triggers a new build.

    *   **Poll SCM:**
        *   **Concept:** Jenkins periodically checks the SCM repository for new changes. If changes are detected since the last check, a build is triggered.
        *   **Syntax (Cron):** Uses cron syntax to define the schedule.
            *   Five fields: `MINUTE HOUR DAY_OF_MONTH MONTH DAY_OF_WEEK`.
            *   `* * * * *`: Every minute.
            *   `H/5 * * * *`: Every 5 minutes (H is a "hash" value, distributes load).
        *   **Step-by-Step (Optional Demo):**
            *   In your `HelloWorldPipeline` job configuration, under **"Build Triggers"**, check **"Poll SCM"**.
            *   **Schedule:** Enter `* * * * *` (for every minute for demo).
            *   Click **"Save"**.
            *   Make a change in GitHub and wait for Jenkins to automatically trigger a build based on the polling interval. (Note: Webhooks are generally preferred over polling to reduce server load).

---

### **Lecture 4: Deploying a Simple HTML Website to EC2 (3 hours)**

**Learning Objectives:**
*   Understand the concept of Jenkins Master-Agent architecture.
*   Configure a new EC2 instance as a Jenkins agent.
*   Write a Declarative Pipeline to deploy a static HTML website to the agent EC2.

**Content & Step-by-Step Guide:**

1.  **Jenkins Master-Agent (Slave) Architecture:**
    *   **Concept:** Jenkins can distribute workloads across multiple machines (agents or slaves) to improve efficiency and scalability.
    *   **Master:** The central Jenkins server that orchestrates jobs and manages agents.
    *   **Agent (Slave/Worker Node):** Machines that run and execute the actual build jobs on behalf of the Master.
    *   **Why use Agents?**
        *   **Offload Workload:** Keeps the Master free for management tasks.
        *   **Scalability:** Easily add more agents as needed.
        *   **Different Environments:** Run jobs on different OS/tools (e.g., Windows for .NET, Linux for Java).
        *   **Security:** Isolate build environments.
    *   **Communication:** Jenkins Master communicates with agents via TCP/IP protocols (default port 50000).

2.  **Prerequisites on EC2 (Target Deployment Server):**
    *   **Step 2.1: Launch a new EC2 Instance for Deployment Target:**
        *   Launch a **new Ubuntu t2.micro** EC2 instance (similar to Lecture 1, Step 1).
        *   **Name:** `Web-Server-Dev`.
        *   **Key pair:** Use the `jenkins-key.pem` created earlier or create a new one.
        *   **Security Group:** Create a **new security group** (e.g., `sg-web-server`).
            *   **Inbound Rules:**
                *   **SSH (Port 22):** Source `My IP`.
                *   **HTTP (Port 80):** Source `Anywhere`.
                *   **Custom TCP (Port 50000):** Source `Jenkins-Master-Security-Group-ID` (to allow communication from Jenkins Master).
        *   Launch the instance and wait for it to be `Running`.

    *   **Step 2.2: Install Apache2 and Git on the Target EC2:**
        *   Connect to `Web-Server-Dev` via SSH.
        *   Update packages: `sudo apt update`.
        *   Install Apache2 and Git: `sudo apt install -y apache2 git`.
        *   Start Apache2: `sudo systemctl start apache2`.
        *   Enable Apache2 on boot: `sudo systemctl enable apache2`.
        *   **Verify Apache:** Open `http://<Web-Server-Dev-Public-IP>` in browser; you should see the default Apache welcome page.

3.  **Configuring `Web-Server-Dev` as a Jenkins Agent (Step-by-Step):**

    *   **Step 3.1: Install Java (JRE) on `Web-Server-Dev`**
        *   `sudo apt update`
        *   `sudo apt install -y openjdk-17-jre-headless`
        *   `java --version` (verify installation)

    *   **Step 3.2: Create a New Jenkins Node/Agent in Jenkins UI**
        *   From Jenkins Dashboard, go to **"Manage Jenkins"** -> **"Nodes"** (or "Manage Nodes and Clouds").
        *   Click **"New Node"**.
        *   **Node name:** `Web-Server-Dev-Agent`.
        *   Select **"Permanent Agent"**.
        *   Click **"Create"**.
        *   **Configure Node:**
            *   **Description:** `Agent for deploying HTML website.`
            *   **Number of executors:** `1` (for simple sequential tasks).
            *   **Remote root directory:** `/home/ubuntu/jenkins-agent` (This is the workspace on the agent).
            *   **Labels:** `dev_agent` (This is crucial; it's how your `Jenkinsfile` will target this agent).
            *   **Usage:** Select `Only build jobs with label expressions matching this node`.
            *   **Launch method:** Select `Launch agent by connecting it to the controller`.
            *   Leave other settings as default. Click **"Save"**.

    *   **Step 3.3: Connect the Agent to Jenkins Master**
        *   After saving the node, Jenkins will display instructions to connect.
        *   **Download `agent.jar`:** Click the link provided on the Jenkins node page to download `agent.jar` to your local machine.
        *   **Transfer `agent.jar` to `Web-Server-Dev`:**
            *   Open a new terminal session.
            *   `scp -i jenkins-key.pem agent.jar ubuntu@<Web-Server-Dev-Public-IP>:/home/ubuntu/jenkins-agent/`
        *   **Run JNLP command on `Web-Server-Dev`:**
            *   SSH back into `Web-Server-Dev`.
            *   Navigate to the agent directory: `cd /home/ubuntu/jenkins-agent`.
            *   Run the command provided by Jenkins UI (it will look something like `java -jar agent.jar -jar -workDir "/home/ubuntu/jenkins-agent" -secret <secret> -url <jenkins-url>/computer/Web-Server-Dev-Agent/`). **Add `&` at the end to run in background:**
                `java -jar agent.jar -jar -workDir "/home/ubuntu/jenkins-agent" -secret <secret> -url <jenkins-url>/computer/Web-Server-Dev-Agent/ &`.
        *   **Verify Connection:** Go back to Jenkins UI -> **"Manage Jenkins"** -> **"Nodes"**. `Web-Server-Dev-Agent` should now show as `Connected`.

    *   **Step 3.4: Configure `sudo NOPASSWD` for Jenkins Agent (Crucial for Deployment)**
        *   For Jenkins to execute `sudo` commands on the `Web-Server-Dev` agent without password prompts (which would make the pipeline hang), you need to configure `sudoers`.
        *   SSH into `Web-Server-Dev`.
        *   Edit sudoers file: `sudo visudo`
        *   Add the following line at the end of the file (assuming agent runs as `ubuntu` user):
            `ubuntu ALL=(ALL) NOPASSWD: /bin/rm, /bin/cp, /bin/systemctl`
            *   (This grants `ubuntu` user passwordless `sudo` for `rm`, `cp`, and `systemctl`. For a real production environment, permissions should be more granular and ideally executed by a dedicated service user, but this simplifies for beginners.)
        *   Save and exit (`:wq`).

4.  **Simple HTML Website in GitHub:**
    *   Ensure your `hello-world-html` GitHub repository (from Lecture 2) contains:
        *   `index.html`:
            ```html
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <title>Hello World Jenkins CI/CD</title>
                <style>
                    body {
                        font-family: Arial, sans-serif;
                        text-align: center;
                        padding-top: 50px;
                        background-color: #f0f0f0;
                        color: #333;
                    }
                    h1 {
                        color: #007bff;
                    }
                    p {
                        font-size: 1.2em;
                    }
                </style>
            </head>
            <body>
                <h1>Hello, Jenkins CI/CD!</h1>
                <p>This page was deployed automatically by Jenkins.</p>
                <p>Version: 1.0</p>
            </body>
            </html>
            ```
        *   `Jenkinsfile` (will be updated in next step).

5.  **CI/CD Pipeline (`Jenkinsfile`) for HTML Deployment:**

    *   **Step 5.1: Update `Jenkinsfile` in GitHub:**
        *   Edit the `Jenkinsfile` in your `hello-world-html` GitHub repository.
        *   Replace its content with the following:
            ```groovy
            pipeline {
                agent { label 'dev_agent' } // This targets your Jenkins agent on Web-Server-Dev-Agent

                stages {
                    stage('Code Checkout') {
                        steps {
                            echo 'Cloning repository...'
                            // Jenkins automatically checks out the SCM into the workspace, no explicit git clone needed here
                        }
                    }
                    stage('Deploy to EC2') {
                        steps {
                            echo 'Deploying application to EC2...'
                            // Clean existing files in Apache web root on the agent EC2
                            sh 'sudo rm -rf /var/www/html/*'

                            // Copy new files from Jenkins workspace to Apache web root
                            // '.' refers to the current workspace directory (where the repo was checked out)
                            sh 'sudo cp -r ./* /var/www/html/' 

                            // Restart Apache to apply changes
                            sh 'sudo systemctl restart apache2'
                            echo 'Deployment complete!'
                        }
                    }
                }

                post {
                    always {
                        echo 'Pipeline finished.'
                    }
                    success {
                        echo 'Deployment successful! Check your website on EC2.'
                        // Optional: Add email notification here (covered in Lecture 5)
                    }
                    failure {
                        echo 'Deployment failed. Please check pipeline logs.'
                        // Optional: Add email notification here (covered in Lecture 5)
                    }
                }
            }
            ```
        *   Commit the changes to GitHub.

    *   **Step 5.2: Trigger Jenkins Pipeline:**
        *   Due to the GitHub Webhook configured in Lecture 3, Jenkins should automatically trigger a build for `HelloWorldPipeline`.
        *   If not, manually trigger it by clicking **"Build Now"** on the `HelloWorldPipeline` job page.

    *   **Step 5.3: Verify Deployment:**
        *   After the Jenkins pipeline completes successfully, open your web browser.
        *   Go to `http://<Web-Server-Dev-Public-IP-Address>` (the public IP of your `Web-Server-Dev` EC2 instance).
        *   You should see your "Hello, Jenkins CI/CD!" HTML page successfully deployed.

6.  **Automated Updates:**
    *   **Demonstrate:** Make a small text change to your `index.html` file directly on GitHub (e.g., change `Version: 1.0` to `Version: 1.1`).
    *   Commit the change.
    *   Observe Jenkins automatically trigger a new build (due to webhook).
    *   After the build completes, refresh your website in the browser. You should see the updated content reflecting the change.

---

### **Lecture 5: Advanced Pipeline Features & Best Practices (3 hours)**

**Learning Objectives:**
*   Implement parameters for dynamic pipeline execution.
*   Use `when` expressions for conditional stage execution.
*   Set up parallel stages to run tasks concurrently.
*   Configure post-build actions, including email notifications.
*   Understand and use `stash` and `unstash` for artifact transfer.
*   Add human input for manual approvals in the pipeline.

**Content & Step-by-Step Guide:**

1.  **Parameters:**
    *   **Concept:** Allows users to input values at runtime, making pipelines flexible and reusable without modifying the `Jenkinsfile` itself.
    *   **Types:** String, Boolean, Choice, Password, etc..
    *   **Step-by-Step: Adding a Choice Parameter for Version**
        *   Edit `Jenkinsfile` in GitHub.
        *   Add a `parameters` block *inside* the `pipeline` block, *before* `agent` or `stages`:
            ```groovy
            pipeline {
                parameters {
                    choice(name: 'APP_VERSION', choices: ['1.0', '1.1', '2.0'], description: 'Select application version to deploy')
                }
                agent { label 'dev_agent' }
                // ... rest of your pipeline
            }
            ```
        *   Modify `index.html` (e.g., change `Version: 1.0` to `Version: ${APP_VERSION}`). *(Note: Shell `echo` or `sed` command in `Jenkinsfile` will be needed to dynamically replace this in `index.html` on the agent side)*
            *   **Simple dynamic `index.html` creation in Jenkinsfile:**
                ```groovy
                stage('Deploy to EC2') {
                    steps {
                        echo "Deploying version ${params.APP_VERSION} to EC2..."
                        sh 'sudo rm -rf /var/www/html/*'
                        sh """
                            sudo tee /var/www/html/index.html > /dev/null <<EOL
                            <!DOCTYPE html>
                            <html lang="en">
                            <head>
                                <meta charset="UTF-8">
                                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                                <title>Hello World Jenkins CI/CD</title>
                                <style>
                                    body {
                                        font-family: Arial, sans-serif;
                                        text-align: center;
                                        padding-top: 50px;
                                        background-color: #f0f0f0;
                                        color: #333;
                                    }
                                    h1 {
                                        color: #007bff;
                                    }
                                    p {
                                        font-size: 1.2em;
                                    }
                                </style>
                            </head>
                            <body>
                                <h1>Hello, Jenkins CI/CD!</h1>
                                <p>This page was deployed automatically by Jenkins.</p>
                                <p>Version: ${params.APP_VERSION}</p>
                            </body>
                            </html>
                            EOL
                        """
                        sh 'sudo systemctl restart apache2'
                        echo 'Deployment complete!'
                    }
                }
                ```
        *   Commit `Jenkinsfile` to GitHub.
        *   In Jenkins, `HelloWorldPipeline` job, click **"Build with Parameters"**. Select a version and build. Verify the version on the deployed website.

2.  **Conditional Stages (`when` expression):**
    *   **Concept:** Allows a stage to execute only if a specified condition is true.
    *   **Common conditions:** `branch`, `environment`, `expression` (Groovy expression).
    *   **Step-by-Step: Deploy to Production Conditionally**
        *   **Prerequisite:** Launch a new EC2 instance `Web-Server-Prod` and configure it as a Jenkins agent (`prod_agent`) similar to Lecture 4, ensuring Apache2 is installed and `sudo NOPASSWD` for `ubuntu` user. Also, open Port 80 in its security group.
        *   Add another choice parameter for deployment environment:
            ```groovy
            pipeline {
                parameters {
                    choice(name: 'DEPLOY_ENV', choices: ['dev', 'prod'], description: 'Select deployment environment')
                }
                // ... other parameters and agent
            ```
        *   Add a new stage for production deployment in `Jenkinsfile`:
            ```groovy
            // ... previous stages
            stage('Deploy to Prod') {
                when {
                    expression { params.DEPLOY_ENV == 'prod' } // Only run if parameter is 'prod'
                    beforeAgent true // Evaluate condition before provisioning agent
                }
                agent { label 'prod_agent' } // Target the production agent
                steps {
                    echo "Deploying version ${params.APP_VERSION} to Production..."
                    sh 'sudo rm -rf /var/www/html/*'
                    sh """
                        sudo tee /var/www/html/index.html > /dev/null <<EOL
                        <!DOCTYPE html>
                        <html lang="en">
                        <head>
                            <meta charset="UTF-8">
                            <meta name="viewport" content="width=device-width, initial-scale=1.0">
                            <title>Hello World Jenkins Prod</title>
                            <style>
                                body {
                                    font-family: Arial, sans-serif;
                                    text-align: center;
                                    padding-top: 50px;
                                    background-color: #f0f0f0;
                                    color: #333;
                                }
                                h1 {
                                    color: #dc3545; /* Red for Prod */
                                }
                                p {
                                    font-size: 1.2em;
                                }
                            </style>
                        </head>
                        <body>
                            <h1>Hello, Jenkins Production!</h1>
                            <p>This page was deployed automatically by Jenkins.</p>
                            <p>Version: ${params.APP_VERSION}</p>
                        </body>
                        </html>
                        EOL
                    """
                    sh 'sudo systemctl restart apache2'
                    echo 'Production deployment complete!'
                }
            }
            ```
        *   Modify the `Deploy to EC2` stage to also use a `when` condition: `when { expression { params.DEPLOY_ENV == 'dev' } }`.
        *   Commit `Jenkinsfile` to GitHub.
        *   Build `HelloWorldPipeline` with `DEPLOY_ENV` as `dev` and then `prod` to observe conditional execution and skip behavior.

3.  **Parallel Stages:**
    *   **Concept:** Allows multiple stages or steps to run concurrently to save time, especially useful for independent tasks like running tests on different environments.
    *   **Step-by-Step: Parallel Test Stages**
        *   Modify `Jenkinsfile`:
            ```groovy
            // ...
            stage('Run Tests') {
                parallel { // This block runs its contained stages concurrently
                    stage('Unit Tests') {
                        steps {
                            echo 'Running Unit Tests...'
                            sh 'echo "Simulating Unit Tests..." && sleep 5' // Simulate work
                        }
                    }
                    stage('Integration Tests') {
                        steps {
                            echo 'Running Integration Tests...'
                            sh 'echo "Simulating Integration Tests..." && sleep 7' // Simulate work
                        }
                    }
                }
            }
            // ...
            ```
        *   Commit `Jenkinsfile` to GitHub.
        *   Run pipeline and observe parallel execution in Jenkins Stage View or Blue Ocean.

4.  **Post Actions:**
    *   **Concept:** Define actions to be taken at the end of the entire pipeline or a specific stage, regardless of success or failure. Useful for cleanup, notifications, or reporting.
    *   **Conditions:** `always`, `success`, `failure`, `unstable`, `changed`.
    *   **Step-by-Step: Email Notifications**

        *   **4.1.1: Configure Jenkins Email Notification:**
            *   Go to **"Manage Jenkins"** -> **"Configure System"**.
            *   Scroll down to **"Email Notification"** (or "Extended E-mail Notification").
            *   **SMTP Server:** `smtp.gmail.com`.
            *   **Default user E-mail suffix:** (Optional, your domain suffix if applicable)
            *   **Use SMTP Authentication:** Check this.
            *   **Username:** Your Gmail address (e.g., `your.email@gmail.com`).
            *   **Password:** **Generate an App Password** from your Google Account security settings. (Go to `myaccount.google.com` -> Security -> App passwords. Generate a new app password for "Mail" and "Other" device). Use this generated password, NOT your regular Gmail password.
            *   **Use SSL/TLS:** Check this.
            *   **SMTP Port:** `465`.
            *   **Test Configuration:** Enter your email in the "Test configuration by sending test email" section and click "Test configuration" to verify.
            *   Click **"Save"**.

        *   **4.1.2: Add Post-Actions to `Jenkinsfile`:**
            *   Modify `Jenkinsfile`:
                ```groovy
                pipeline {
                    // ...
                    post {
                        always {
                            echo 'Pipeline run completed.'
                        }
                        success {
                            echo 'Pipeline succeeded! Sending success email.'
                            mail to: 'your.email@example.com',
                                 subject: 'Jenkins Pipeline SUCCESS: Hello World App Deployment',
                                 body: "The Hello World application was successfully deployed. Build: ${env.BUILD_NUMBER}"
                        }
                        failure {
                            echo 'Pipeline failed! Sending failure email.'
                            mail to: 'your.email@example.com',
                                 subject: 'Jenkins Pipeline FAILED: Hello World App Deployment',
                                 body: "The Hello World application deployment failed. Build: ${env.BUILD_NUMBER}\n\nCheck logs for details.",
                                 attachLog: true // Attaches console output to email
                        }
                    }
                }
                ```
            *   Commit `Jenkinsfile` to GitHub.
            *   Run pipeline: once successful, check your email. Introduce an error (e.g., wrong `rm` command) and re-run to test failure notification.

5.  **`stash` and `unstash`:**
    *   **Concept:** `stash` saves a set of files from the workspace on one agent to Jenkins controller, and `unstash` retrieves them on potentially a different agent or a different stage's workspace. Useful for transferring artifacts without committing them back to SCM.
    *   **Step-by-Step (Illustrative, complex for HTML, better for compiled apps):**
        *   Imagine a "Build" stage on one agent compiles a `MyApp.zip`.
        *   After `sh 'zip -r MyApp.zip .'`, you would add:
            `stash name: 'my-app-artifact', includes: 'MyApp.zip'`.
        *   Then, in a "Deploy" stage on a different agent:
            `unstash 'my-app-artifact'`.
            This would make `MyApp.zip` available in the current agent's workspace.
        *   *(For this simple HTML app, `cp` command is sufficient as files are small and directly in SCM.)*

6.  **Human Input for Approval:**
    *   **Concept:** Introduce a pause in the pipeline that requires manual approval to proceed. Ideal for critical steps like production deployments.
    *   **Step-by-Step:**
        *   In your `Jenkinsfile`, add an `input` step before your `Deploy to Prod` stage:
            ```groovy
            stage('Approve Production Deployment') {
                when {
                    expression { params.DEPLOY_ENV == 'prod' }
                    beforeAgent true
                }
                steps {
                    script {
                        input message: 'Proceed with production deployment?', submitter: 'admin, qa_lead',
                            ok: 'Approve Deployment',
                            parameters: [string(name: 'REASON', defaultValue: 'Approved by QA', description: 'Reason for approval')]
                    }
                    echo "Deployment approved for reason: ${REASON}"
                }
            }
            // ... then your 'Deploy to Prod' stage
            ```
        *   Commit `Jenkinsfile`.
        *   Run pipeline with `prod` environment. Observe the pipeline pause and wait for input. Click "Proceed" and provide a reason to continue.

---

### **Lecture 6: Reusability and Organization in Jenkins Pipelines (3 hours)**

**Learning Objectives:**
*   Understand the concept of Shared Libraries for code reusability.
*   Implement a simple Shared Library and use it in a pipeline.
*   Learn to manage credentials securely in Jenkins.
*   Set up Role-Based Access Control (RBAC) for Jenkins users.

**Content & Step-by-Step Guide:**

1.  **Shared Libraries:**
    *   **Concept:** A powerful feature that allows you to define reusable Groovy code (functions) in an external Git repository, which can then be called from any of your Jenkinsfiles. This promotes the **DRY (Don't Repeat Yourself)** principle.
    *   **Structure:** A Shared Library repository typically contains a `vars` directory. Each `.groovy` file in `vars` defines a global variable (or function) that can be accessed in Pipelines. A common convention is to define a `call()` method inside these `.groovy` files.

    *   **Step-by-Step: Implementing a Simple Deployment Shared Library**

        *   **1.1: Create a GitHub Repository for Shared Library:**
            *   Create a **new public GitHub repository** (e.g., `https://github.com/your-username/jenkins-shared-libs`).
            *   Inside this repo, create a folder named `vars`.
            *   Inside `vars`, create a file named `simpleHtmlDeploy.groovy`.
            *   Add the following content to `simpleHtmlDeploy.groovy`:
                ```groovy
                // vars/simpleHtmlDeploy.groovy
                def call(String targetIp, String appVersion) {
                    echo "Executing deployment function for target: ${targetIp} with version: ${appVersion}"
                    // This is simplified: in a real scenario, you'd transfer files
                    // For this demo, we use the agent's current workspace
                    sh 'sudo rm -rf /var/www/html/*'
                    sh """
                        sudo tee /var/www/html/index.html > /dev/null <<EOL
                        <!DOCTYPE html>
                        <html lang="en">
                        <head>
                            <meta charset="UTF-8">
                            <meta name="viewport" content="width=device-width, initial-scale=1.0">
                            <title>Hello World Shared Lib</title>
                            <style>
                                body {
                                    font-family: Arial, sans-serif;
                                    text-align: center;
                                    padding-top: 50px;
                                    background-color: #f0f0f0;
                                    color: #333;
                                }
                                h1 {
                                    color: #28a745; /* Green for Shared Lib Demo */
                                }
                                p {
                                    font-size: 1.2em;
                                }
                            </style>
                        </head>
                        <body>
                            <h1>Hello from Shared Library!</h1>
                            <p>Deployed to ${targetIp} automatically.</p>
                            <p>Version: ${appVersion}</p>
                        </body>
                        </html>
                        EOL
                    """
                    sh 'sudo systemctl restart apache2'
                    echo "Deployment to ${targetIp} completed by Shared Library function."
                }
                ```
            *   Commit the file to your `jenkins-shared-libs` repository.

        *   **1.2: Configure Jenkins to Use the Shared Library:**
            *   Go to **"Manage Jenkins"** -> **"System"** (or "Configure System").
            *   Scroll down to **"Global Pipeline Libraries"**.
            *   Click **"Add"**.
            *   **Name:** `my-shared-lib`.
            *   **Default version:** `main` (or `master`, your default branch name).
            *   **Retrieval Method:** Select **"Modern SCM"** -> **"Git"**.
            *   **Project Repository:** Paste the HTTPS URL of your `jenkins-shared-libs` repository (`https://github.com/your-username/jenkins-shared-libs.git`).
            *   Click **"Save"**.

        *   **1.3: Use the Shared Library in `Jenkinsfile`:**
            *   Edit your `HelloWorldPipeline`'s `Jenkinsfile` in GitHub.
            *   Add `@Library('my-shared-lib') _` at the very top of the script, before `pipeline {`:
                ```groovy
                @Library('my-shared-lib') _

                pipeline {
                    // ... your pipeline definition
                }
                ```
            *   Replace the `steps` content in your `Deploy to EC2` and `Deploy to Prod` stages with calls to `simpleHtmlDeploy`:
                ```groovy
                stage('Deploy to EC2') {
                    when { expression { params.DEPLOY_ENV == 'dev' } }
                    agent { label 'dev_agent' }
                    steps {
                        script { // 'script' block is needed to call shared library functions
                            simpleHtmlDeploy("${env.NODE_NAME_OR_IP}", "${params.APP_VERSION}") // Assuming a way to get IP, or hardcode for demo
                        }
                    }
                }
                stage('Deploy to Prod') {
                    when { expression { params.DEPLOY_ENV == 'prod' } }
                    agent { label 'prod_agent' }
                    steps {
                        script {
                            simpleHtmlDeploy("${env.NODE_NAME_OR_IP}", "${params.APP_VERSION}") // Assuming a way to get IP, or hardcode for demo
                        }
                    }
                }
                ```
                *(Note: `${env.NODE_NAME_OR_IP}` is a placeholder. You'd typically need to get the agent's IP via a shell command like `sh 'hostname -I'` and store it in an environment variable, then pass that.) For simplicity, you can hardcode the Public IP of your Dev/Prod EC2 in the `Jenkinsfile` for the `simpleHtmlDeploy` function parameter.*
            *   Commit `Jenkinsfile`.
            *   Run pipeline and verify that deployment happens via the shared library function.

2.  **Credentials Management (Securely using Tokens/Secrets):**
    *   **Concept:** Jenkins provides a secure store for sensitive information like passwords, API tokens, and SSH keys. This prevents hardcoding secrets in `Jenkinsfile`.
    *   **Types:** Secret Text, Username and Password, SSH Username with Private Key.

    *   **Step-by-Step: Storing a GitHub Personal Access Token (PAT)**
        *   **2.1: Generate GitHub Personal Access Token (PAT):**
            *   Go to your GitHub account -> **"Settings"** -> **"Developer settings"** -> **"Personal access tokens"** -> **"Tokens (classic)"**.
            *   Click **"Generate new token"**.
            *   **Note:** Give it a meaningful name (e.g., `Jenkins-PAT`).
            *   **Scopes:** For basic repo operations, select `repo` (all sub-options). For read-only, select `public_repo`.
            *   Click **"Generate token"**. **Copy the generated token immediately** (it won't be shown again).

        *   **2.2: Add PAT to Jenkins Credentials:**
            *   Go to **"Manage Jenkins"** -> **"Credentials"**.
            *   Click **"Jenkins"** scope -> **"Global credentials (unrestricted)"**.
            *   Click **"Add Credentials"**.
            *   **Kind:** Select **"Secret text"**.
            *   **Secret:** Paste your copied GitHub PAT.
            *   **ID:** `github-pat` (This is the ID you'll reference in `Jenkinsfile`).
            *   **Description:** `GitHub Personal Access Token for Jenkins`.
            *   Click **"Create"**.

        *   **2.3: Using Credentials in `Jenkinsfile` (Basic Example):**
            *   If you needed to clone a *private* repository within a step (which Jenkins SCM handles for the main repo, but if you need to clone another one):
                ```groovy
                stage('Clone Private Repo') {
                    steps {
                        withCredentials([string(credentialsId: 'github-pat', variable: 'GH_TOKEN')]) {
                            // GH_TOKEN will contain the secret PAT for this block
                            sh 'git clone https://oauth2:${GH_TOKEN}@github.com/your-username/your-private-repo.git'
                        }
                    }
                }
                ```
                *(Explain `withCredentials` block and `variable` binding)*.
            *   Alternatively, for environment variables:
                ```groovy
                pipeline {
                    environment {
                        MY_GITHUB_TOKEN = credentials('github-pat') // This makes it available as MY_GITHUB_TOKEN
                    }
                    // ... stages
                    stage('Use Token') {
                        steps {
                            sh 'echo "Using token (first few chars): ${MY_GITHUB_TOKEN.substring(0, 5)}..."'
                            // Use the token for Git commands directly if needed
                        }
                    }
                }
                ```
            *   Commit and test.

3.  **Role-Based Access Control (RBAC):**
    *   **Concept:** Define different roles with specific permissions and assign users to those roles. This controls who can do what in Jenkins (e.g., who can build, configure, or administer).
    *   **Step-by-Step:**

        *   **3.1: Install Role-based Authorization Strategy Plugin:**
            *   Go to **"Manage Jenkins"** -> **"Plugins"** -> **"Available plugins"**.
            *   Search for **"Role-based Authorization Strategy"**.
            *   Install it. Restart Jenkins if prompted.

        *   **3.2: Enable Role-based Strategy:**
            *   Go to **"Manage Jenkins"** -> **"Configure Global Security"**.
            *   Under **"Authorization"** section, select **"Role-Based Strategy"**.
            *   Click **"Save"**. A new menu item "Manage and Assign Roles" will appear under "Manage Jenkins".

        *   **3.3: Create and Assign Roles:**
            *   Go to **"Manage Jenkins"** -> **"Manage and Assign Roles"**.
            *   **3.3.1: Manage Roles:**
                *   Click **"Manage Roles"**.
                *   **Add "Viewer" Role:**
                    *   Under **"Global Roles"**, add a role named `viewer`.
                    *   Check only `Overall/Read` permission.
                *   **Add "Developer" Role:**
                    *   Add a role named `developer`.
                    *   Check `Job/Build`, `Job/Read`, `Job/Discover`.
                *   Click **"Save"**.
            *   **3.3.2: Assign Roles:**
                *   Click **"Assign Roles"**.
                *   Under **"Global Roles"**, add the users you want to assign roles to.
                *   Add `admin` (your initial admin user) and ensure it has `admin` role checked.
                *   Add `Authenticated Users` (for general logged-in users) and assign them the `viewer` role.
                *   **Create a new user (e.g., `testuser`)** by going to `Manage Jenkins` -> `Users` -> `Create User`.
                *   Go back to **"Manage and Assign Roles"** -> **"Assign Roles"**.
                *   Add `testuser` to the `developer` role.
                *   Click **"Save"**.

        *   **3.4: Test RBAC:**
            *   Open an incognito/private browser window.
            *   Log in as `testuser`.
            *   Observe that `testuser` can only see and build jobs, but cannot configure Jenkins or other administrative tasks.
            *   Log in as the initial `admin` user to confirm full access.

---

**Course Conclusion:**
Students will have learned the foundational concepts of CI/CD, how to install and manage Jenkins, build different types of jobs/pipelines, deploy a simple application, and implement advanced features for pipeline automation, reusability, and security. This solid base prepares them for more complex DevOps tasks and advanced Jenkins functionalities.
