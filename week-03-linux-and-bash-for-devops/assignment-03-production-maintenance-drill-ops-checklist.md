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

![screenshot1](screenshots/ss3.3.1.JPG)

---

#### Screenshot 2 — Output of `ip a`

![screenshot2](screenshots/ss3.3.2.JPG)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![screenshot3](screenshots/ss3.3.3.JPG)

---

#### Screenshot 4 — Output of `sudo ufw status`

![screenshot4](screenshots/ss3.3.4.JPG)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

The output of sudo ss -tulpen showed port 80 in the LISTEN state on 0.0.0.0, confirming Nginx accepts HTTP traffic on all network interfaces.
---

**2. What proves SSH is active on port 22?**

The same command showed sshd listening on port 22, confirming the server is ready to accept SSH connections.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No, I found only the required ports such as 22 for SSH and 80 for HTTP. Keeping unnecessary ports closed reduces security risk.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![screenshot1](screenshots/ss3.3.5.JPG)
---

#### Screenshot 2 — Output of `sudo nginx -t`

![screenshot2](screenshots/ss3.3.6.JPG)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![screenshot3](screenshots/ss3.3.7.JPG)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

The website may become unavailable or users may receive errors such as connection refused or 502 errors, resulting in downtime.

---

**2. What's your basic rollback plan?**

I would restore the last working Nginx configuration, validate it with sudo nginx -t, and restart or reload Nginx after confirming the syntax is correct.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![screenshot1](screenshots/ss3.3.8.JPG)

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![screenshot2](screenshots/ss3.9.JPG)

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![screenshot3](screenshots/ss3.3.10.JPG)
---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No recent errors were found in the Nginx error log, which indicates that Nginx was operating normally during the check.

---

**2. If there were no errors, what does that indicate about the system?**

It indicates that requests were being processed normally and no server-side failures were recorded during the monitoring period.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes, the curl requests appeared in the access log, proving that the requests reached Nginx and were successfully processed and logged.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![screenshot1](screenshots/ss3.3.11.JPG)

---

#### Screenshot 2 — Output of `free -h`

![screenshot2](screenshots/ss3.3.12.JPG)

---

#### Screenshot 3 — Output of `df -h`

![screenshot3](screenshots/ss3.3.13.JPG)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![screenshot4](screenshots/ss3.3.14.JPG)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

None of the resources appeared critical, but disk usage should be monitored because logs, deployments and updates can gradually consume available storage.

---

**2. What happens if disk becomes 100% full in a production server?**

Applications may fail to write files or logs, deployments can fail, and services may crash or refuse to start, potentially causing downtime.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![screenshot1](screenshots/ss3.3.15.JPG)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

Add your screenshot here.

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![screenshot3](screenshots/ss3.3.17.JPG)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I verified the files in /var/www/html, searched for my personalized “Deployed by” text, and confirmed the updated application loaded through the server’s public IP.

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

![screenshot3](screenshots/ss3.3.19.JPG)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The failure was caused by intentionally removing the semicolon from the try_files directive, which created an invalid Nginx configuration.

---

**2. How did you fix the issue?**

I restored the missing semicolon, validated the configuration with sudo nginx -t, and restarted or reloaded Nginx after the test passed.

---

**3. How can you avoid this kind of issue in real production systems?**

Always run nginx -t before reloading Nginx, review configuration changes, use version control and test changes in a staging environment first.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![screenshot3](screenshots/ss3.3.20.JPG)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![screenshot3](screenshots/ss3.3.19.JPG)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The application broke because /var/www/html was replaced with an empty directory, so Nginx could not find the React build files.

---

**2. How did you fix the issue and restore the application?**

I removed the empty directory, restored the original deployment directory from backup, and confirmed recovery with an HTTP 200 OK response.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

I would use backups, version-controlled deployments, automated pipelines with rollback support, staging tests and post-deployment monitoring.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH keys are much harder to guess and avoid sharing reusable passwords between users.

---

**2. Why should only required ports be open on a production server?**

Closing unused ports reduces the attack surface and limits the number of services exposed to the internet.

---

**3. Why is it important for Nginx to be enabled on boot?**

It ensures Nginx starts automatically after a reboot, reducing downtime and avoiding manual recovery.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Attackers could gain unauthorized access, steal data, misuse cloud resources or create unexpected charges.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Unused resources can continue generating charges and increase unnecessary security and maintenance risks.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

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

- [x] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [x] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [x] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [x] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [x] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [x] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [x] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [x] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [x] Full Name visible in all required screenshots
- [x] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*