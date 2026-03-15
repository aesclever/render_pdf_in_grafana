# render_pdf_in_grafana

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
