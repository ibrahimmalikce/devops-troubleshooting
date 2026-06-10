# Apache Tomcat Startup Failure — AlmaLinux Troubleshooting Runbook

## Environment
- OS: AlmaLinux (Dell PowerEdge R420)
- Service: Apache Tomcat (systemd)
- Java: OpenJDK 17.0.19

## Symptoms
- `systemctl start tomcat` fails immediately
- `status=1/FAILURE` on Main PID
- Duration only ~8ms (JVM never fully starts)

## Diagnostic Steps

### 1. Check service status
```bash
sudo systemctl status tomcat
```
Showed: `Main process exited, code=exited, status=1/FAILURE`

### 2. Check bin/ directory
```bash
sudo ls -la /opt/tomcat/bin/
```
Showed: `Permission denied` — admin user couldn't read the directory.

### 3. Check catalina.out logs
```bash
sudo cat /opt/tomcat/logs/catalina.out
```
Revealed two JVM errors:
- `Could not find or load main class Djava.awt.headless=true`
- `Invalid initial heap size: -Xms512M-Xmx1024M`

### 4. Inspect the systemd service file
```bash
sudo cat /etc/systemd/system/tomcat.service
```
Found the misconfigurations.

## Root Causes

| # | Issue | Buggy Config | Fixed Config |
|---|-------|-------------|--------------|
| 1 | bin/ not readable by admin | `drwxr-x---` | `chmod 755 /opt/tomcat/bin` |
| 2 | Missing `-` in JAVA_OPTS | `JAVA_OPTS=Djava.awt.headless=true` | `JAVA_OPTS=-Djava.awt.headless=true` |
| 3 | Missing space in heap flags | `-Xms512M-Xmx1024M` | `-Xms512M -Xmx1024M` |

## Fix Applied

```bash
# Fix bin/ permissions
sudo chmod 755 /opt/tomcat/bin
sudo find /opt/tomcat/bin -name "*.sh" -exec chmod +x {} \;

# Edit service file
sudo nano /etc/systemd/system/tomcat.service
```

Corrected lines in service file:
```ini
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
Environment="JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom"
```

```bash
# Reload and restart
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl enable tomcat

# Verify
curl http://localhost:8080
```

## Outcome
Tomcat running successfully — `active (running)`, enabled on boot.

## Key Lesson
When Tomcat fails instantly on startup, always check `catalina.out` first.
The systemd status only shows the exit code — the real error is in the log.
