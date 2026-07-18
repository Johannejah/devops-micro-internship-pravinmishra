# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

Add your screenshot here.

---

#### Screenshot 2 — Output of `ip a`

Add your screenshot here.

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

Add your screenshot here.

---

#### Screenshot 4 — Output of `sudo ufw status`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

Write your answer here.

---

**2. What proves SSH is active on port 22?**

There are two clear proofs that SSH is active on port 22. Locally on the virtual machine, running sudo systemctl status ssh displays an active and running daemon, and checking netstat/ss commands shows sshd listening on port 22. Remotely, the definitive proof is our ability to establish a successful terminal session from our local machine using ssh -i <key.pem> ubuntu@<public-ip>, which wouldn't initialize if port 22 wasn't open and listening.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No unexpected open ports were found active during the configuration. Outside of standard system loopback operations, the scan and listening sockets only showed port 22 actively open for our secure administrative SSH connection and port 80 actively open for Nginx to serve the EpicReads web app traffic. This minimal footprint is ideal as it keeps the attack surface small and aligns perfectly with cloud security best practices.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo nginx -t`

Add your screenshot here.

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart during a production update, the web server can experience two critical issues depending on how it failed. If it crashed entirely, the service drops, resulting in an immediate outage where users face "Connection Refused" or timeout errors when accessing the site. If it simply rejected the new configuration during a reload, it may continue running on the old configuration; however, this leaves the deployment in a broken state where new code updates are stalled, and any subsequent server reboots will cause a total service failure.

---

**2. What's your basic rollback plan?**

My basic rollback plan follows a structured, three-step safety protocol to minimize downtime:

1. Validate and Revert Configurations: Immediately run sudo nginx -t to identify the syntax error. If it cannot be fixed within seconds, copy the last known working backup file back into /etc/nginx/sites-available/default.

2. Restore the Build Directory: If the error is caused by a corrupt application build, quickly swap the Nginx root directory path back to the previous stable build folder or a static maintenance page.

3. Force a Clean Restart: Execute sudo systemctl restart nginx to bring the server back to a stable running state, ensuring the public URL is functional before troubleshooting the new deployment files offline.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

Add your screenshot here.

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No, the Nginx error log (/var/log/nginx/error.log) did not show any recent errors during my check. An empty error log, or one without recent entries, simply means that the Nginx master and worker processes are running smoothly, the configuration syntax is valid, and the server has not encountered any internal crashes, permission denials, or broken upstream connections while attempting to serve the application files.

---

**2. If there were no errors, what does that indicate about the system?**

The absence of errors indicates that the web server environment is completely healthy, stable, and correctly configured. It proves that Nginx has full read permissions to the React production build directory (/home/ubuntu/my-react-app/build), the network ports are binding successfully without conflict, and the system is operating exactly as intended under normal conditions.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes, the curl requests were clearly visible in the Nginx access logs (/var/log/nginx/access.log), showing the local or remote IP address, the timestamp, a HTTP GET / request, and a successful 200 OK status response. This visibility definitively proves that the entire network traffic loop is open and functional; it demonstrates that requests successfully pass through the AWS firewall/security groups, reach the EC2 instance, are accepted by Nginx, and receive a proper response from the application.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

Add your screenshot here.

---

#### Screenshot 2 — Output of `free -h`

Add your screenshot here.

---

#### Screenshot 3 — Output of `df -h`

Add your screenshot here.

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Memory (RAM) is the most critical resource to watch closely right now. Running JavaScript-heavy modern applications like a React build process or hosting background node modules uses a significant portion of a standard micro-instance's memory footprint. While a high CPU load spikes temporarily during builds and drops immediately after, low available memory risks trigger the Linux Out-Of-Memory (OOM) killer, which will abruptly terminate the Nginx or Node process and take the site offline.

---

**2. What happens if disk becomes 100% full in a production server?**

When a production server's disk hits 100% capacity, it triggers an immediate system crisis. Nginx and the operating system will no longer be able to write to essential access or error logs, causing services to crash or reject incoming traffic. Additionally, database transactions will fail, package managers cannot install updates, cron jobs stall, and administrative functions (like creating SSH temporary session keys) break entirely—effectively locking you out of the machine and causing a severe application outage.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

Add your screenshot here.

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

Add your screenshot here.

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

Write your answer here.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

Add your screenshot here.

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

Add your screenshot here.

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

Write your answer here.

---

**2. How did you fix the issue?**

Write your answer here.

---

**3. How can you avoid this kind of issue in real production systems?**

Write your answer here.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

Add your screenshot here.

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

Write your answer here

---

**2. How did you fix the issue and restore the application?**

Write your answer here.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

Write your answer here.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

Write your answer here.

---

**2. Why should only required ports be open on a production server?**

Write your answer here.

---

**3. Why is it important for Nginx to be enabled on boot?**

Write your answer here.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Write your answer here.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Write your answer here.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`__________________________`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*