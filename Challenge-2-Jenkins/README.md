# Jenkins CI/CD Troubleshooting Challenge

This repository documents my hands-on practice diagnosing automated pipeline failures, resolving webhook misconfigurations, handling disk space constraints, and understanding full CI/CD architectural flows.

---

## 🛠️ The Scenario
A developer pushes code changes to a remote GitHub repository. Normally, the automated Jenkins orchestration agent triggers a build instantaneously. Today, the code is pushed successfully, but no pipeline activity occurs.

---

## 🔍 Troubleshooting Workflow

### Step 1: External Diagnostic Checks
To isolate whether the issue originates from the source provider (GitHub) or the automation server (Jenkins), I trace the payload delivery path:
1. Navigate to GitHub Repository **Settings** -> **Webhooks**.
2. Select the specific Jenkins webhook and inspect the **Recent Deliveries** panel.
3. Analyze the HTTP Status Codes:
   * 🟢 **200 OK:** GitHub successfully dispatched the webhook payload, and the endpoint responded. The breakdown is internal to Jenkins configuration.
   * 🔴 **4xx Error (400/403/404):** Indicates authentication issues, faulty routing, or expired CSRF crumb tokens on Jenkins.
   * 🔴 **5xx Error / Connection Timeout:** Jenkins is completely offline, down, or unreachable behind a restricted firewall/security group.

### Step 2: Extracting Jenkins Logs
If the webhook returns a 200 OK but no build processes launch, I inspect Jenkins internal subsystems:
* **GitHub Hook Log:** (`Manage Jenkins -> System Log -> GitHub Hook Log`) To verify if Jenkins correctly evaluated the incoming JSON payload and found a matching job trigger specification.
* **System Log:** To scan for global server failures, deadlocks, or threading faults.

---

## 💾 Resolving "No space left on device" Errors

When a Jenkins runner exhausts its local physical volume storage, pipelines freeze or terminate immediately.

### Critical Recovery Commands
* **Wipe Local Workspaces:** Purge dynamic, temporary workspaces from old builds: `rm -rf /var/lib/jenkins/workspace/*`
* **Docker Garbage Collection:** Clear out heavy, dangling container layers caching on the build host machine:
  ```bash
  docker system prune -a --volumes -f