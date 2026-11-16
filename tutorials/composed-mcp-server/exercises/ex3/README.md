# Exercise 3 — Connect, Govern & Analyze an MCP Server using SAP Integration Suite

## Overview

Your MCP Server is deployed, published, and governed. Now it is time to connect an AI agent, observe how the traffic policies protect it in practice, and monitor the consumption patterns — all from within SAP Integration Suite.

In this exercise, you will connect the MCP Inspector as an MCP client using the OAuth credentials from [Exercise 2](../ex2/README.md), make live tool calls against the supply risk data, and deliberately test the Surge Protection and Quota policies you configured. You will then use Monitor Message Processing to inspect how each request — successful or rejected — was handled by the policy pipeline.

> [!IMPORTANT]
> You will need the **MCP URL** from Ex. 2.4 and the **OAuth credentials** (Token URL, Key, Secret) from Ex. 2.7 to complete this exercise. Have them ready before you begin.

---

## Ex. 3.1 — Connect via MCP Inspector

Any AI agent or MCP client can consume the MCP Server. For this workshop, your instructor has provided a hosted **MCP Inspector** (MCP Client) URL; open it in your browser.

### Step 1 — Add Your MCP Server

Click **Add Servers → + Add manually**.

![MCP Inspector — Add Servers dropdown](../../resources/screenshots/ex2-step8-add-servers-menu.png)

In the **Add server** dialog, fill in:

| Field | Value |
|---|---|
| Server ID | `plantsupplyrisk-mcp-XX` (replace XX with your assigned participant number) |
| Transport | `streamable-http` |
| URL | Your MCP URL from Ex. 2.4 (e.g. `https://<virtual-host>/mcp-plantsupplyrisk-XX`) |

Click **Add**

![Add server dialog — Server ID, Transport, URL filled in](../../resources/screenshots/ex2-step8-add-server-dialog.png)

### Step 2 — Configure OAuth Authentication

On the server card, click **Settings**.

Scroll to the bottom of the Settings dialog and click **OAuth Settings** to expand it.

![Server Settings — OAuth Settings section highlighted](../../resources/screenshots/ex2-step8-server-settings-oauth.png)

Configure the following fields:

| Field | Value |
|---|---|
| **Enterprise-managed authorization** | Leave **unchecked** |
| **Client ID** | Your **Key** from Ex. 2.7 |
| **Client Secret** | Your **Secret** from Ex. 2.7 |
| **Request refresh token** | **Uncheck** (checked by default) |
| **Revoke tokens on clear** | **Uncheck** (checked by default) |

![OAuth Settings — Client ID and Client Secret filled in](../../resources/screenshots/ex2-step8-oauth-settings-credentials.png)

Scroll down within OAuth Settings to **Token URL override** and enter your **Token URL** from Ex. 2.7. Leave **Authorization URL override** blank.

![OAuth Settings — Token URL override filled in](../../resources/screenshots/ex2-step8-oauth-settings-token-url.png)

Close Settings.

### Step 3 — Connect and Make Initial Tool Calls

Toggle the server switch to **Connected**. The server card turns green confirming a successful connection.

![MCP Inspector — plantsupplyrisk-mcp-xx Connected](../../resources/screenshots/ex2-step8-connected.png)

In the top navigation bar, click **Tools**. You will see the 4 operations exposed from your OpenAPI spec. Click **Retrieve a list of assessment** (`get_bestrun_assessment`) — the parameter form opens in the center pane. No parameters are required — click **Execute Tool**.

![MCP Inspector — get_bestrun_assessment selected with Execute Tool button](../../resources/screenshots/ex2-step8-tools-list.png)

The Results pane shows all plants with their IDs (Berlin, Munich, London, etc.).

![MCP Inspector — Assessment results showing plant list](../../resources/screenshots/ex2-step8-assessment-results.png)

Pick `id` — **`my.custom~0010`** from the results and click **Retrieve a list of address risks of an assessment** (`get_bestrun_assessment_id_addressRisks`). Enter the `id` value in the **id** field.

![MCP Inspector — id parameter field filled in for address risks tool](../../resources/screenshots/ex2-step8-address-risk-id.png)

Click **Execute Tool** — this returns the live risk scores for that plant's location.

![MCP Inspector — Address risk results for a plant](../../resources/screenshots/ex2-step8-address-risk-results.png)

> **Tip:** The data you see here — plant IDs, risk factors, risk scores — is exactly what an AI agent will receive when it calls these tools autonomously to answer a procurement question.

> [!NOTE]
> The tool calls you make in this step generate **successful** message entries in Monitor Message Processing. These will appear alongside the policy violation messages you create in Ex. 3.2, giving you a complete picture when you analyze consumption in Ex. 3.3.

### Troubleshooting

| Issue | Fix |
|---|---|
| Cannot connect | Verify the MCP URL matches the one shown in Ex. 2.4 |
| 401 Unauthorized | Verify Client ID, Client Secret, and Token URL are entered correctly in OAuth Settings |
| No tools listed | Ensure the correct tools were selected during Ex. 2.2 Step 4 |
| Issuer mismatch error | Ensure **Enterprise-managed authorization** is **unchecked** in OAuth Settings |

---

## Ex. 3.2 — Govern: Observe Traffic Policies in Action

With the MCP Inspector connected and initial tool calls made, you will now observe both Traffic Management policies in action. Before triggering any policy violations, you will first enable trace logging — this ensures that every subsequent request, whether successful or rejected, is captured in full detail for inspection in Ex. 3.3.

### Step 1 — Enable Trace Logging

In SAP Integration Suite, go to **Monitor → Integrations and APIs**. Set **Runtime** to **Integration Cell**.

Click the **Manage Integration Content** tile.

![Monitor Overview — Manage Integration Content tile](../../resources/screenshots/ex2-step9-monitor-overview-content.png)

Set **Runtime** to **Integration Cell**. In the **Integration Content** list, click on **`plantsupplyrisk_XX`**.

In the right panel, scroll to **Log Configuration** and set the **Log Level** to **Trace**.

![Manage Integration Content — plantsupplyrisk_XX Log Configuration set to Trace](../../resources/screenshots/ex2-step9-manage-content-trace.png)

> [!NOTE]
> Trace logging captures full policy execution details and request/response payloads for every message processed. It is automatically reset after approximately 1 hour. Enabling it now ensures both the Surge Protection and Quota violations you trigger next are recorded with full detail.

Return to MCP Inspector before proceeding to the next step.

---

### Step 2 — Trigger Surge Protection

Recall the Surge Protection policy configured in Ex. 2.3:
- **Limit:** 3 calls within any 9-second window
- **On breach:** `429 Too Many Requests` — request is aborted before reaching the backend

In MCP Inspector, click **Retrieve a list of assessment** (`get_bestrun_assessment`) and click **Execute Tool** four times as fast as possible — do not wait for each response before clicking again.

![MCP Inspector — Surge Protection error response](../../resources/screenshots/ex3-step2-surge-error.png)

The first three calls succeed. The fourth call within the 9-second window returns:

```json
{
  "error": {
    "code": "surgeProtectionLimitExceeded",
    "message": "Surge Protection limit exceeded for maximum number of calls"
  }
}
```

> [!NOTE]
> Wait approximately 10 seconds and click **Execute Tool** again — the request succeeds. Surge Protection resets after the window expires. This distinguishes it from Quota, which does not reset until the next day.

---

### Step 3 — Trigger Quota Management

Recall the Quota Management policy configured in Ex. 2.3:
- **Limit:** 15 calls per day per OAuth Client ID
- **On breach:** `429 Too Many Requests` — request is aborted before reaching the backend
- **Tracked by:** `${context.authn.getClientID()}` — your subscription has its own independent counter

> [!IMPORTANT]
> To trigger Quota without being blocked by Surge Protection first, space each call at least **10 seconds apart**. Calls rejected by Surge Protection do not count toward the quota — only successful calls decrement the counter.

Make tool calls on `get_bestrun_assessment`, waiting 10 seconds between each. The successful calls from Ex. 3.1 already count toward your quota. Continue until the 16th successful call of the day, which returns:

```json
{
  "error": {
    "code": "quotaLimitExceeded",
    "message": "Quota limit exceeded"
  }
}
```

![MCP Inspector — Quota exceeded error response](../../resources/screenshots/ex3-step2-quota-error.png)

> [!NOTE]
> Unlike Surge Protection, the Quota limit does not reset within the workshop session — it resets at the start of the next quota window (based on the Start Time you configured in UTC). This is intentional: Quota governs total daily consumption per agent subscription, not momentary burst rate.

With both policies triggered, the quota is now exhausted. No further successful tool calls are possible in this quota window — proceed directly to Ex. 3.3 to analyze what was captured.

---

## Ex. 3.3 — Analyze: Monitor MCP Consumption

Every request processed by your MCP Server — successful or rejected — is recorded as a message in Monitor Message Processing. With trace logging enabled from Ex. 3.2, each message carries full policy execution detail. In this step you will inspect all three message types side by side and understand exactly what the policy pipeline did for each.

### Step 1 — Open Monitor Message Processing

In SAP Integration Suite, go to **Monitor → Integrations and APIs**.

Click the **Monitor Message Processing** tile.

![Monitor Overview — Monitor Message Processing tile highlighted](../../resources/screenshots/ex2-step9-monitor-overview-messages.png)

### Step 2 — View All Message Types

Set **Runtime** to **Integration Cell**. You will see a list of messages for `plantsupplyrisk_XX` — both completed and failed. The list reflects everything that happened across Ex. 3.1 and Ex. 3.2:

| Message Status | Origin | What happened |
|---|---|---|
| **Completed** | Ex. 3.1 tool calls | Request passed all policies and reached the backend |
| **Failed** | Surge Protection breach (Ex. 3.2) | Request was aborted at the Surge Protection step |
| **Failed** | Quota breach (Ex. 3.2) | Request was aborted at the Quota step |

Click on a **Completed** message row to open the detail panel.

The panel shows:
- **Status** — Message processing completed successfully
- **Processing Time**
- **Logs** — Log Level: Trace

Click **Trace**

![Monitor Message Processing — completed message with detail panel](../../resources/screenshots/ex2-step9-message-detail.png)

### Step 3 — Inspect Run Steps for a Successful Message

Click **MCP Server Model**.

The **Run Steps** panel shows each policy component and its execution time — **Authentication**, **Authorization 1**, **Surge Protection**, and **Quota** — all highlighted green, overlaid on the Policy Model diagram. This confirms the request passed through every policy and reached the backend.

![Monitor — Run Steps for a successful message](../../resources/screenshots/ex3-step3-run-steps-success.png)

### Step 4 — Inspect Run Steps for a Surge Protection Failure

Go back to the message list and click on a **Failed** message that corresponds to the Surge Protection breach.

In the **Run Steps** panel, the flow stops at the **Surge Protection** node — highlighted in red. Authentication and Authorization 1 completed successfully, but Surge Protection aborted the request before it reached Quota or the backend.

![Monitor — Run Steps for a Surge Protection failure](../../resources/screenshots/ex3-step3-run-steps-surge.png)

### Step 5 — Inspect Run Steps for a Quota Failure

Click on a **Failed** message that corresponds to the Quota breach.

In the **Run Steps** panel, the flow progresses further — Authentication, Authorization 1, and Surge Protection all completed — but stops at the **Quota** node. This reflects the order of the policy chain: Quota is only reached by requests that have already passed the burst check.

![Monitor — Run Steps for a Quota failure](../../resources/screenshots/ex3-step3-run-steps-quota.png)

> [!NOTE]
> Comparing the three Run Steps diagrams side by side is the clearest way to understand how the policy chain works. Each policy acts as a gate — a request reaches the next gate only if it passes the current one. The Run Steps diagram shows exactly where each request stopped and why.

---

## Summary

In this exercise you:

- Connected the **MCP Inspector** to your MCP Server using OAuth credentials and verified all four tools are accessible
- Enabled **Trace logging** to capture full policy execution detail for all subsequent requests
- Triggered **Surge Protection** — observed the `surgeProtectionLimitExceeded` error when exceeding 3 calls within a 9-second window, and confirmed the policy resets after the window expires
- Triggered **Quota Management** — observed the `quotaLimitExceeded` error when the daily call allowance per OAuth Client ID was exhausted
- Used **Monitor Message Processing** to inspect all three message types — successful, Surge Protection failure, and Quota failure — and compared the **Run Steps** diagrams to see exactly which policy fired for each

The full MCP Server lifecycle is now complete: composed from enterprise APIs in Exercise 1, deployed and governed in Exercise 2, and connected, tested, and analyzed in Exercise 3.

---

[Back to Overview](../../../../README.md)
