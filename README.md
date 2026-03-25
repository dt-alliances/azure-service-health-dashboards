# Azure Service Health Dashboards for Dynatrace

A collection of Dynatrace dashboards (v21 JSON format) for monitoring Azure health signals across Service Health, Resource Health, Advisor Recommendations, and Security alerts.

These dashboards query Azure Activity Log data ingested into Dynatrace via the Azure log forwarder, using `log.source` to filter each category.

## Dashboards

### Overview Dashboard

**File:** [`azure-health-overview-dashboard-v1.json`](azure-health-overview-dashboard-v1.json)

A central dashboard providing a single-pane-of-glass view across all four Azure health log sources. Includes:

- Key KPI tiles from each area (Service Health, Resource Health, Recommendations, Security)
- Pie charts for Recommendations by Category and Security by Severity
- Combined timeline showing all 4 log sources on a single chart, color-coded by source
- Markdown link tiles to drill into each detailed dashboard

> **Setup:** After importing all dashboards, replace the placeholder dashboard IDs (`SERVICE_HEALTH_DASHBOARD_ID`, `RESOURCE_HEALTH_DASHBOARD_ID`, `RECOMMENDATIONS_DASHBOARD_ID`, `SECURITY_DASHBOARD_ID`) in the markdown link tiles with the actual IDs from each imported dashboard's URL.

---

### Service Health Dashboard

**File:** [`azure-servicehealth-dashboard-v1.json`](azure-servicehealth-dashboard-v1.json)
**Log Source:** `log.source == "ServiceHealth"`

Monitors Azure service incidents, planned maintenance, health advisories, and security advisories.

| Feature | Details |
|---------|---------|
| **Variables** | Status, EventType, Region, Service, ImpactType |
| **KPIs** | Total Events, Active Issues, Resolved, Incidents, Maintenance, High Impact (HIR), Sensitive Events |
| **Charts** | Pie charts by Status, Incident Type, Service, Region, Impact Type |
| **Tables** | Event Summary, Event Details with communication |
| **Timeline** | Events over time grouped by incident type |

---

### Service Health Enhanced Dashboard (with Entity Correlation)

**File:** [`azure-servicehealth-dashboard-enhanced.json`](azure-servicehealth-dashboard-enhanced.json)
**Log Source:** `log.source == "ServiceHealth"`

Extended version of the Service Health dashboard with an **Entity Correlation** section that maps Azure health events to Dynatrace-monitored entities. Includes everything from the standard dashboard plus:

- **Correlated Entities** - Count of Dynatrace entities with active Azure health alerts
- **Regions Affected** - Number of Azure regions currently impacted
- **Correlated Azure VMs** - Table of entities receiving health event alerts
- **Problems During Health Events** - Davis problems correlated with Azure health events in the same time window
- **Azure Health vs Dynatrace Problems Timeline** - Side-by-side comparison of both event streams
- **Correlation Events Detail** - Raw view of events created by the correlation workflow

> **Requires:** The [`correlation-workflow.json`](correlation-workflow.json) automation workflow to be deployed in Dynatrace.

---

### Resource Health Dashboard

**File:** [`azure-resourcehealth-dashboard-v1.json`](azure-resourcehealth-dashboard-v1.json)
**Log Source:** `log.source == "ResourceHealth"`

Monitors individual Azure resource availability and health status changes.

| Feature | Details |
|---------|---------|
| **Variables** | Status, HealthStatus, Cause, EventType, Level |
| **KPIs** | Total Events, Active, Unavailable, Degraded, Platform Initiated, User Initiated |
| **Charts** | Pie charts by Health Status, Cause, Event Type, Level |
| **Tables** | Resource Health Summary, Event Details |
| **Timeline** | Events over time grouped by health status |

---

### Recommendations Dashboard

**File:** [`azure-recommendations-dashboard-v1.json`](azure-recommendations-dashboard-v1.json)
**Log Source:** `log.source == "Recommendation"`

Displays Azure Advisor recommendations for improving security, cost, performance, and availability.

| Feature | Details |
|---------|---------|
| **Variables** | Status, Category, Impact, Level, Location |
| **KPIs** | Total, Active, High Impact, Security, Cost, Performance, High Availability |
| **Charts** | Pie charts by Category, Impact, Status, Level, Location |
| **Tables** | Recommendations Summary, Recommendation Details with resource links |
| **Timeline** | Recommendations over time grouped by category |

---

### Security Dashboard

**File:** [`azure-security-dashboard-v1.json`](azure-security-dashboard-v1.json)
**Log Source:** `log.source == "Security"`

Monitors Azure Security Center alerts including compromised entities and remediation guidance.

| Feature | Details |
|---------|---------|
| **Variables** | Status, Severity, AttackedResourceType, Level, Location |
| **KPIs** | Total Alerts, Active Alerts, High/Medium/Low Severity, VM Attacks, Unique Compromised Entities |
| **Charts** | Pie charts by Severity, Attacked Resource Type, Status, Compromised Entity, Location |
| **Tables** | Security Alerts Summary, Alert Details with remediation steps |
| **Timeline** | Alerts over time grouped by severity |

---

## Correlation Workflow

**File:** [`correlation-workflow.json`](correlation-workflow.json)

A Dynatrace Automation workflow that runs every 6 hours to correlate Azure Service Health events with Dynatrace-monitored Azure VM entities.

**How it works:**
1. Queries active Service Health events from logs
2. Filters for VM-related services (Virtual Machines, Compute, Dedicated Host)
3. Maps Azure regions to Dynatrace region identifiers
4. Finds Azure VM entities in impacted regions
5. Creates custom events (`AVAILABILITY_EVENT` or `CUSTOM_INFO`) on matched entities

**Custom event properties created:**
- `azure.health.trackingId`, `azure.health.service`, `azure.health.region`
- `azure.health.incidentType`, `azure.health.stage`, `azure.health.isHighImpact`
- `azure.health.impactStartTime`, `azure.health.portalLink`
- `source: "Azure Service Health"`, `correlation.type: "azure-health-entity"`

This enables **Davis AI** to correlate Azure health events with detected problems on affected entities.

## Prerequisites

- Azure Activity Logs forwarded to Dynatrace (via Azure log forwarder or OpenPipeline)
- Log data available under `log.source` values: `ServiceHealth`, `ResourceHealth`, `Recommendation`, `Security`
- For entity correlation: Azure VM entities monitored by Dynatrace

## Import Instructions

1. In Dynatrace, go to **Dashboards**
2. Click **Upload** and select the desired JSON file
3. The dashboard will be created with all tiles, variables, and layouts pre-configured
4. For the **Overview dashboard**, update the markdown link placeholders with actual dashboard IDs after importing all dashboards
5. For the **Enhanced dashboard**, deploy the `correlation-workflow.json` via **Workflows > Upload/Import**
