# Security Monitoring With Tripwire
Tripwire is a host-based intrusion detection system (HIDS) used monitor changes in critical files and directories. This project involves installing and configuring Tripwire on a Linux system, setting up monitoring for critical files and automating alerts for unauthorized changes

### Steps to Complete the Project:

1. **Install and Configure Tripwire**.
2. **Set up Monitoring for Critical Files and Directories**.
3. **Automate Alerts for Unauthorized Changes**.


### Step 1: Install and Configure Tripwire on a Linux System

#### Step 1.1: Install Tripwire

For **Ubuntu/Debian** systems:

```bash
sudo apt update
sudo apt install tripwire
```

For **RHEL/CentOS** systems:

```bash
sudo yum install tripwire
```

#### Step 1.2: Initialize Tripwire Configuration

1. During installation, you will be prompted to configure Tripwire. Select **"Yes"**.
2. Provide a **local passphrase** (used to secure the configuration and database files).
3. Initialize the Tripwire database.

Run the following command:

```bash
sudo tripwire --init
```

- This creates a baseline of the current system. Any changes to monitored files and directories after initialization will be flagged.

#### Step 1.3: Customize the Tripwire Configuration File

Tripwire uses the `twpol.txt` file as its policy file. Modify it to include the directories and files you want to monitor.

1. Edit the policy file:

   ```bash
   sudo nano /etc/tripwire/twpol.txt
   ```

2. Add or modify the directories and files under **Rule Definitions**. For example:

   ```plaintext
   # Monitor system binaries
   /bin         -> $(SEC_BIN),
   /sbin        -> $(SEC_BIN),

   # Monitor critical system configuration files
   /etc/passwd  -> $(SEC_CRIT),
   /etc/shadow  -> $(SEC_CRIT),
   /etc/ssh     -> $(SEC_CRIT),
   ```

3. Save and update the policy file:

   ```bash
   sudo twadmin --create-polfile /etc/tripwire/twpol.txt
   ```

4. Reinitialize the database to apply changes:

   ```bash
   sudo tripwire --init
   ```

---

### Step 2: Set up Monitoring for Critical Files and Directories

#### Step 2.1: Verify the Policy

Check the current policy for monitored files:

```bash
sudo tripwire --check
```

- This command scans the system and compares the current state with the baseline. Any changes will be flagged in the output.

#### Step 2.2: Tailor Monitoring to Your Use Case

Here’s an example of files and directories typically monitored in production systems:

- **System binaries**: `/bin`, `/sbin`, `/usr/bin`, `/usr/sbin`
- **Critical configuration files**: `/etc/passwd`, `/etc/shadow`, `/etc/ssh/sshd_config`
- **Application-specific files**: `/var/www/html` (for web servers), `/opt/custom_app` (custom applications)

Update `twpol.txt` accordingly.

#### Step 2.3: Test Monitoring

Make a small, deliberate change to a monitored file (e.g., `/etc/hosts`) to test detection:

1. Modify the file:

   ```bash
   echo "127.0.0.1 malicious-site.com" | sudo tee -a /etc/hosts
   ```

2. Run the check again:

   ```bash
   sudo tripwire --check
   ```

- The output should flag the change to `/etc/hosts`.

3. Revert the change:

   ```bash
   sudo nano /etc/hosts
   ```

4. Rebuild the Tripwire database to accept legitimate changes:

   ```bash
   sudo tripwire --update --twrfile /var/lib/tripwire/report/<report_file>.twr
   ```

---

### Step 3: Automate Alerts for Unauthorized Changes

To automate alerts for unauthorized changes, use Tripwire's built-in reporting capabilities and integrate it with an email or log management system.

#### Step 3.1: Enable Email Notifications

1. Edit Tripwire’s configuration file:

   ```bash
   sudo nano /etc/tripwire/twcfg.txt
   ```

2. Configure email settings (example for `sendmail`):

   ```plaintext
   MAILMETHOD = SENDMAIL
   MAILPROGRAM = "/usr/sbin/sendmail -oi -t"
   ```

3. Save the changes and update the configuration file:

   ```bash
   sudo twadmin --create-cfgfile -o /etc/tripwire/tw.cfg /etc/tripwire/twcfg.txt
   ```

4. Test email functionality by triggering a Tripwire check:

   ```bash
   sudo tripwire --check | mail -s "Tripwire Report" admin@example.com
   ```

#### Step 3.2: Schedule Periodic Checks with Cron

Automate periodic checks by adding a cron job.

1. Open the cron editor:

   ```bash
   sudo crontab -e
   ```

2. Add the following job to run a check daily and email the results:

   ```bash
   0 2 * * * /usr/sbin/tripwire --check | mail -s "Daily Tripwire Report" admin@example.com
   ```

#### Step 3.3: Integrate with SIEM (Optional)

For centralized logging and monitoring, send Tripwire reports to a Security Information and Event Management (SIEM) system like Splunk or ELK (Elastic Stack).

1. Configure Tripwire to log to `/var/log/tripwire.log`.
2. Set up log forwarding using tools like `rsyslog` or `Filebeat`.

---

### Final Step: Document the Tripwire Setup

Documenting the process ensures the setup is reproducible and helps train your team.

---

#### **Tripwire Setup Documentation Template**

**1. Overview**  
Tripwire monitors file integrity and alerts administrators to unauthorized changes. This document outlines installation, configuration, and automation.

**2. Installation**  
- Install Tripwire:
  ```bash
  sudo apt install tripwire
  ```
- Initialize the database:
  ```bash
  sudo tripwire --init
  ```

**3. Configuration**  
- Edit the policy file (`/etc/tripwire/twpol.txt`) to monitor critical files:
  ```plaintext
  /bin         -> $(SEC_BIN),
  /etc/passwd  -> $(SEC_CRIT),
  ```
- Reinitialize the database:
  ```bash
  sudo tripwire --init
  ```

**4. Automation**  
- Schedule daily checks using `cron`:
  ```bash
  0 2 * * * /usr/sbin/tripwire --check | mail -s "Daily Tripwire Report" admin@example.com
  ```

**5. Testing**  
- Make a deliberate change to a file and verify detection:
  ```bash
  sudo tripwire --check
  ```


### Training the Team

1. **Hands-On Practice**: Provide team members with a sandbox environment to practice installing and configuring Tripwire.
2. **Review Reports**: Teach the team how to interpret Tripwire reports.
3. **Response Protocols**: Establish a protocol for responding to unauthorized changes flagged by Tripwire.

By following this step-by-step guide, you will successfully set up a robust file integrity monitoring system with Tripwire, ensuring your system's security and equipping your team with the skills to manage it effectively.
