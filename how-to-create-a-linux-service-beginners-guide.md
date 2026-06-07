# How to Create a Service: A Beginner's Guide

## What Is a Service?

A **service** is a program that runs in the background on a computer — it starts automatically, keeps running without anyone logged in, and does its job quietly. Examples include a web server (serving websites), a database engine, or a scheduled task runner.

Think of a service like a light switch in a hotel hallway: it turns on when the building starts up, stays on all day, and doesn't need someone standing there to keep it running.

---

## Why Use a Service?

| Without a Service | With a Service |
|---|---|
| You must manually start the program | Starts automatically on boot |
| Stops when you log out | Keeps running in the background |
| No automatic restart on crash | Can restart itself on failure |
| Hard to manage logs | Logs are centralized and organized |

---

## Step 1: Write Your Program

Before creating a service, you need a program to run. Let's use a simple Python script as an example.

Create a file called `my_service.py`:

```python
import time
import logging

# Set up logging so we can see what the service is doing
logging.basicConfig(
    filename='/var/log/my_service.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

def main():
    logging.info("Service started!")
    while True:
        logging.info("Service is running...")
        time.sleep(60)  # Wait 60 seconds before logging again

if __name__ == "__main__":
    main()
```

**What this does:** Every 60 seconds, the script writes a message to a log file. Simple — but it demonstrates the idea perfectly.

---

## Step 2: Create a systemd Service File (Linux)

On Linux, **systemd** is the standard tool for managing services. You define your service in a plain text file.

Create a new file at `/etc/systemd/system/my_service.service`:

```ini
[Unit]
Description=My First Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/youruser/my_service.py
Restart=always
User=youruser

[Install]
WantedBy=multi-user.target
```

**Breaking it down:**

- `Description` — a human-readable name for your service
- `After=network.target` — wait for the network to be ready before starting
- `ExecStart` — the exact command that starts your program
- `Restart=always` — if the program crashes, restart it automatically
- `User` — which system user runs the service
- `WantedBy=multi-user.target` — start this service when the system reaches normal multi-user mode (i.e., normal boot)

---

## Step 3: Enable and Start Your Service

Open a terminal and run these commands one by one:

```bash
# 1. Tell systemd to re-read all service files
sudo systemctl daemon-reload

# 2. Enable the service so it starts on every boot
sudo systemctl enable my_service

# 3. Start the service right now
sudo systemctl start my_service
```

---

## Step 4: Check That It's Running

```bash
# See the status of your service
sudo systemctl status my_service
```

You should see something like:

```
● my_service.service - My First Service
     Loaded: loaded (/etc/systemd/system/my_service.service; enabled)
     Active: active (running) since ...
```

**"active (running)"** means your service is alive and working. 🎉

---

## Step 5: View the Logs

```bash
# See recent log output from your service
sudo journalctl -u my_service -n 50
```

This shows the last 50 lines of log output. It's your window into what the service is actually doing.

---

## Common Commands Cheat Sheet

| Command | What It Does |
|---|---|
| `systemctl start my_service` | Start the service |
| `systemctl stop my_service` | Stop the service |
| `systemctl restart my_service` | Restart the service |
| `systemctl status my_service` | Check if it's running |
| `systemctl enable my_service` | Auto-start on boot |
| `systemctl disable my_service` | Remove auto-start |
| `journalctl -u my_service` | View logs |

---

## Troubleshooting Tips

1. **Service won't start?** Run `systemctl status my_service` — it often shows the exact error.
2. **Permission denied errors?** Make sure the `User=` in your service file has access to the script and log files.
3. **Changes not taking effect?** Always run `sudo systemctl daemon-reload` after editing the `.service` file.
4. **Script path wrong?** Double-check the full path in `ExecStart` — use `which python3` to find the right Python path.

---

## Summary

Creating a service involves just three things:
1. **A program** to run
2. **A service definition file** that tells the system how to run it
3. **A few commands** to register and start it

Once your service is running, the system manages it for you — restarting it on failure, launching it on boot, and keeping logs. That's the power of turning a script into a service.

---

*Happy building! If something doesn't work, the logs almost always tell you why.*
