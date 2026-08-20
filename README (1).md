# IT Helpdesk Lab — osTicket on Ubuntu Server (LAMP)

A self-hosted support ticketing system built from scratch on a Linux server, configured to mirror a real Tier 1 / Tier 2 helpdesk and integrated with an [Active Directory home lab](https://github.com/esaintuny-lang) to simulate end-to-end IT support workflows.

---

## Overview

This project stands up **osTicket** — an open-source ticketing platform — on a dedicated **Ubuntu Server** virtual machine running a full **LAMP stack** (Linux, Apache, MariaDB, PHP). The system is configured with departments, help-topic routing, and SLA plans that model how a real service desk operates, then exercised with support tickets that reference users and issues from a companion Active Directory environment.

The goal: demonstrate the complete helpdesk skill set — Linux server administration, web/database service deployment, and the ticket lifecycle (intake → triage → resolution → documentation → closure) — in one connected lab.

---

## Architecture

| Component | Details |
| :---- | :---- |
| Hypervisor | Oracle VirtualBox |
| Ticketing host | Ubuntu Server (dedicated VM, bridged networking) |
| Web server | Apache 2.4 |
| Database | MariaDB |
| Runtime | PHP |
| Application | osTicket v1.18.4 |
| Network | Bridged adapter, static/DHCP LAN IP reachable from host and other lab VMs |

The ticketing server runs as a **standalone member server**, separate from the Domain Controller — matching how these services are separated in production.

---

## Build Summary

**1\. Provisioned the server**

- Installed Ubuntu Server on a dedicated VM  
- Configured bridged networking so the helpdesk is reachable across the lab LAN  
- Updated the system and applied security patches

**2\. Deployed the LAMP stack**

sudo apt install apache2 \-y

sudo apt install mariadb-server \-y

sudo apt install php php-cli php-mysql php-gd php-mbstring php-xml php-curl php-intl php-apcu libapache2-mod-php \-y

sudo mysql\_secure\_installation

**3\. Created the application database**

CREATE DATABASE osticket;

CREATE USER 'osticketuser'@'localhost' IDENTIFIED BY '\<password\>';

GRANT ALL PRIVILEGES ON osticket.\* TO 'osticketuser'@'localhost';

FLUSH PRIVILEGES;

**4\. Installed and configured osTicket**

- Downloaded and extracted osTicket into the Apache web root  
- Created an Apache virtual host pointing at the osTicket directory  
- Enabled the site and `mod_rewrite`, disabled the default site  
- Completed the web installer and connected it to the database  
- Hardened the install: locked down `ost-config.php` permissions and removed the `/setup` directory

---

## Helpdesk Configuration

The system was configured to reflect a real IT service desk rather than default out-of-the-box settings.

### Departments

Created to mirror the Organizational Units in the Active Directory lab:

- **IT Support**  
- **Help Desk**  
- **Network**

### Help Topics (with routing)

Each help topic routes tickets to the correct department automatically:

| Help Topic | Routes To |
| :---- | :---- |
| Password Reset | IT Support |
| New Hire Onboarding | IT Support |
| Software Installation | Help Desk |
| Network / WiFi Issue | Network |

Default topics were disabled to keep the configuration intentional and clean.

### SLA Plans

Response-time targets to model ticket prioritization:

| Plan | Grace Period |
| :---- | :---- |
| Critical | 4 hours |
| High Priority | 8 hours |
| Standard | 24 hours |

---

## Integration with the Active Directory Lab

The tickets in this system deliberately reference the [Active Directory environment](https://github.com/esaintuny-lang), tying the two projects into one continuous support workflow:

| Ticket Scenario | Connects To |
| :---- | :---- |
| User locked out of domain account | AD account unlock / password reset in ADUC |
| New hire onboarding | `CREATE_USERS.ps1` — bulk AD account provisioning |
| Employee offboarding | `OFFBOARD_USER.ps1` — disable account, strip groups |
| Client cannot reach network / no DNS | DHCP scope and DNS troubleshooting on the DC |

This mirrors how a real helpdesk operates: tickets come in, and resolving them requires acting on the identity and network infrastructure behind them.

---

## Sample Ticket — Full Lifecycle

**Ticket \#367803 — "User abirk locked out of domain account"**

A complete Tier 1 ticket demonstrating the full resolution workflow:

1. **Intake** — Ticket created via phone, routed to IT Support through the Password Reset help topic, assigned to an agent.  
2. **Diagnosis** — User reported inability to log into their domain workstation; account locked after repeated failed login attempts.  
3. **Customer response** — User notified that their account was being worked on.  
4. **Internal resolution note** — Documented the technical fix for the record:  
     
   > Verified user identity via callback. Unlocked `abirk` account in Active Directory Users & Computers (esaintuny.com domain). Reset password and set "user must change password at next logon." Confirmed `abirk` can authenticate — `whoami` returns `esaintuny\abirk`. Ticket resolved.  
     
5. **Closure** — Ticket marked Resolved with the full thread documented.

This single ticket shows both the **customer-facing** communication and the **behind-the-scenes technical work**, referencing the actual AD user from the companion lab.

---

## Skills Demonstrated

- **Linux server administration** — Ubuntu Server install, package management, service configuration, log analysis, permissions hardening  
- **Web & database services** — Apache virtual hosts, MariaDB database/user creation and privilege management, PHP  
- **Application deployment** — installing and configuring a production web application end to end, including security cleanup  
- **Helpdesk operations** — department structure, ticket routing, SLA design, and the full ticket lifecycle  
- **Cross-system troubleshooting** — connecting ticket resolution back to Active Directory identity and network infrastructure

---

## Screenshots

*(Add your captured screenshots here)*

- `screenshots/departments.png` — Department structure  
- `screenshots/help-topics.png` — Help topics with routing  
- `screenshots/sla-plans.png` — SLA plans  
- `screenshots/ticket-367803-thread.png` — Full ticket lifecycle thread  
- `screenshots/ticket-queue.png` — Ticket queue

---

## Related Projects

- [**Active Directory & PowerShell Home Lab**](https://github.com/esaintuny-lang) — Windows Server AD DS, DHCP, DNS, and a 3-script PowerShell identity-lifecycle suite (create / offboard / report). The helpdesk tickets in this project reference users and issues from that environment.

