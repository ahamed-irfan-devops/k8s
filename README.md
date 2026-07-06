Markdown

\# Kubernetes Troubleshooting Challenge: CrashLoopBackOff Simulation



This repository documents my hands-on practice simulating, diagnosing, and resolving a `CrashLoopBackOff` state inside a local Kubernetes cluster using Minikube.



\---



\## 🛠️ The Scenario

A Node.js application (`node-app`) was successfully deployed to the cluster, but users are unable to access it. Checking the pod statuses revealed that the container is caught in a crash loop:



```text

NAME                          READY   STATUS             RESTARTS   AGE

node-app-64cb9756bc-plf9k     0/1     CrashLoopBackOff   6          5m

🔍 Troubleshooting Workflow

Step 1: Inspecting the Cluster Environment

To analyze the lifecycle and lifecycle events of the failing pod, I ran the following command to check its operational health:



Bash

kubectl describe pod -l app=node-app

Observation: The Events block showed repeating cycles of Container created -> Container started -> Back-off restarting failed container. This verified that Kubernetes was successfully pulling the image and spawning the runtime environment, but the internal application process was exiting immediately.



Step 2: Extracting Application Logs

To determine why the application was shutting down, I retrieved the logs from the active and previous failed iterations:



Bash

kubectl logs -l app=node-app

kubectl logs -l app=node-app --previous

Output:



Plaintext

Starting app...

CRASHING NOW!

💡 Root Cause \& Resolution

Root Cause

The testing manifest (test-crash.yaml) was explicitly configured with an entrypoint script designed to terminate with a failure exit code (exit 1) right after initialization:



YAML

command: \["sh", "-c", "echo 'Starting app...'; sleep 2; echo 'CRASHING NOW!'; exit 1"]

Resolution

Configured Persistence: Modified the container startup string to simulate a persistent server background process that doesn't close automatically:



YAML

command: \["sh", "-c", "echo 'Starting app...'; echo 'Application is now running perfectly!'; sleep infinity"]

Applied Changes: Updated the cluster resource configuration using:



Bash

kubectl apply -f test-crash.yaml

Verification: Ran kubectl get pods to confirm that the old pod was terminated cleanly and replaced with a stable instance displaying a Running status with 0 restarts.



🧰 Tech Stack \& Tools Used

Kubernetes (k8s) — Container Orchestration Engine



Minikube — Local Multi-Platform Kubernetes Engine (Docker driver runtime)



Git \& GitHub — Version Control and Repository Management

