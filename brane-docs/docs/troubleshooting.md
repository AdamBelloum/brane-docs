# Troubleshooting Common Issues 

_Last updated: October 2025_  
**Audience:** System administrators, developers testing locally  
**Purpose:** Explain how explain how to solve comon issues 
---



During the setup and workflow execution, you may encounter some issues. Below are
common errors and their solutions.

## Docker Exporter Not Supported Error

Error:
```
ERROR: Docker exporter is not supported for the docker driver.
```

**Solution**: 
Ensure that the Docker configuration is set to use the containerd image store. Modify the /etc/docker/daemon.json file and add:

```json
{
   ”experimental” : true,
   ”features” : {
     ”containerd” : true
   }
}
```

Restart Docker:

```bash
sudo systemctl restart docker
```

## Linking Error: Cannot Find sqlite3

Error:
``` /usr/bin/ld: cannot find -lsqlite3
```
Solution: Install the libsqlite3-dev package:

```bash
sudo apt−get installlibsqlite3 −dev
```

## Failed to Run Workflow Error

Error:

```
ERROR: Failed to run workflow
```

Caused by:
Failed to plan workflow

Solution: This error may occur due to the incorrect use of the --remote argument.
Remove the argument from your comman

---

You can proceed to 
[Build Your First Package ?](developer-guide.md)
[Run Your First Workflow ?](getting-started.md)
[? Back to Table of Contents](brane-docs-index.md)

