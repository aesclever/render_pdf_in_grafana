# Render PDF Grafana on Windows Host

A meaty, multi-part project to render a browser-based dashboard → capture panel images → compose a document. 

This is a meaty, multi-part project. 

---

**Why this stack specifically?**

The problem reduces to: *render a browser-based dashboard → capture panel images → compose a document*. Every piece of the stack maps to one of those three steps:

1. **TestData datasource** — eliminates the external database variable. Grafana's built-in TestData source generates realistic random-walk time-series and accepts inline CSV for tabular data. No PostgreSQL, no Prometheus setup needed.

2. **grafana-image-renderer (standalone binary)** — as of September 2025, the plugin form was deprecated and the Image Renderer now exists only as a service deployed separately alongside Grafana. On Windows, you run it as a background service via NSSM (a free Windows service wrapper).

3. **mahendrapaipuri/grafana-dashboard-reporter-app** — this plugin exposes the reporter as a custom API endpoint of Grafana, meaning authenticated access is guaranteed by Grafana's own auth system — no separate web server to secure.

---

**The three phases in the guide:**

- **Phase 1** — Install Grafana, provision a TestData datasource, and deploy a dashboard with a time-series panel, a multi-series comparison chart, and a sortable table panel (all via JSON provisioning files).
- **Phase 2** — Download the renderer Windows binary, configure it with an auth token, wrap it as a Windows service, and point `grafana.ini` at it.
- **Phase 3** — Drop the reporter plugin zip into Grafana's plugins folder, allow the unsigned plugin in `grafana.ini`, enable it in the UI, then call the PDF endpoint via browser or PowerShell (including a `&includePanelDataID=3` parameter to append raw tabular data from the table panel).

The troubleshooting table at the bottom covers the most common Windows-specific gotchas (Chromium sandboxing, unsigned plugin warnings, token mismatches).


Start → ![HERE](WindowsIndex.html)

---
---

# Render PDF Grafana on Ubuntu Host

The guide has been fully ported to Ubuntu 24.04. Here's a summary of every first-principles substitution made — the *why* behind each change:

---

**Service management: NSSM → systemd**
On Windows, NSSM was needed to wrap the renderer binary as a managed service. On Linux, `systemd` is the native init system — you write a `.service` unit file instead. This also gives you proper dependency ordering (`After=network.target`), journald log integration, and security hardening directives (`NoNewPrivileges`, `PrivateTmp`) for free.

**Installation: MSI → APT repo**
Grafana publishes a signed APT repository. This is strictly better than a binary download on Ubuntu — it hooks into the system package manager, handles upgrades with `apt-get upgrade`, and verifies package integrity via GPG automatically.

**Config paths: `C:\Program Files\...` → FHS-standard Linux paths**
Linux follows the Filesystem Hierarchy Standard: config lives in `/etc/grafana/`, runtime data in `/var/lib/grafana/`, logs in `/var/log/grafana/`. The provisioning mechanism is identical — only the paths change.

**Service user: implicit → explicit `useradd --system`**
On Ubuntu you explicitly create a locked, no-login system account (`grafana-renderer`) and `chown` the renderer directory to it. This enforces least-privilege — the renderer process cannot write anywhere outside its working directory. The `chown` step on the provisioning directory is similarly critical, since the `grafana` system user must be able to read those files.

**Chromium dependencies: bundled → must be declared**
The renderer ships its own Chromium binary, but on Ubuntu it needs shared system libraries (X11, NSS, GTK, etc.) that aren't installed by default on a server image. The `apt-get install` block in Step 4a resolves this — it's the most common failure point on a fresh Ubuntu Server install.

**Automation: PowerShell → curl + cron**
`curl` with a Bearer token is the idiomatic Linux equivalent of `Invoke-WebRequest`. A `crontab` entry replaces Windows Task Scheduler for scheduled PDF exports.

Start → ![HERE](UbuntuIndex.html)
