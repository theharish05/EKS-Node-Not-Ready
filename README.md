# Incident Report: Worker Node Exhaustion

This report documents the resolution of a critical cluster issue where a worker node stopped accepting workloads due to running out of available disk space.

## What Happened
The cluster's single worker node became unstable because its storage disk reached full capacity. 

When a server running Kubernetes runs out of hard drive space, it triggers a protective state called **DiskPressure**. To protect itself from crashing completely, the server automatically stops allowing new applications to deploy and begins shutting down lower-priority tasks.

## The Problem (The Catch-22 Loop)
A major challenge occurred during this incident:
* The disk was too full to unpack or start any new software containers.
* Because of this, when we tried to deploy an automated cleanup script through the standard cluster commands, the server instantly blocked and rejected it.

The cleanup tools could not run because the disk was full, and the disk remained full because the cleanup tools could not run.

## How We Recovered the Node
To break this deadlock, we had to bypass the standard cluster management tools entirely and talk directly to the underlying cloud infrastructure provider.

### 1. Direct Infrastructure Access
Instead of using standard cluster deployment commands, we used the cloud provider's secure remote management terminal (AWS Systems Manager) to send a command straight to the operating system kernel of the server.

### 2. Immediate File Truncation
We executed a target command to instantly shrink the problematic files back to a size of zero bytes. By wiping the data without deleting the file paths, the operating system instantly reclaimed all the lost storage space.

### 3. Automatic Recovery
As soon as the server's background metrics detected that the storage drive was empty and healthy again, the system automatically cleared the error flags and returned to a fully functional status.

## Future Recommendations
* **Proactive Alerts:** Configure automated notification systems to alert engineers when server storage space crosses 75% capacity, allowing team intervention before an automated lockout occurs.
* **Automated Log Cleaners:** Maintain background system services that routinely sweep and compress application runtime logs before they grow large enough to cause disk starvation.
