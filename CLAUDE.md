# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Git-backed content for the **Kafka → EDA → Automation Orchestrator** demo. AAP sources both the EDA rulebook and the Controller playbooks directly from this repo — nothing is pasted into the UI manually.

**Target flow:**
```
EDA Event Stream (HTTP POST)
  → EDA rulebook (rulebooks/rulebook.yml)
  → run_job_template: "Orchestrator Webhook Forward"
  → playbooks/forward-to-orchestrator.yml
  → POST /api/v1/webhooks/demo
  → Orchestrator workflow: LLM → approval → hello-world AAP job
```

## Repo Structure

```
rulebooks/
  rulebook.yml        # EDA project source — event stream → run_job_template
playbooks/
  forward-to-orchestrator.yml   # Controller project source — POSTs to Orchestrator webhook
  hello_world.yml               # Controller project source — triggered by Orchestrator n3 node
```

## AAP Projects

Two separate AAP projects point at this repo:

| AAP Project | Playbook dir | Used by |
|---|---|---|
| Orchestrator Demo Playbooks | `playbooks/` | "Orchestrator Webhook Forward" and "Demo Hello World" job templates |
| Orchestrator Demo Rulebook | `rulebooks/` | EDA rulebook activation |

## Key Variables

### forward-to-orchestrator.yml
Fetches a fresh Orchestrator JWT at runtime — no static token baked in.

| Variable | Source |
|---|---|
| `orchestrator_url` | Job template extra vars |
| `orchestrator_username` | AAP credential |
| `orchestrator_password` | AAP credential |
| `orchestrator_event` | EDA `run_job_template` job_args |
| `orchestrator_topic` | EDA `run_job_template` job_args |

### hello_world.yml
| Variable | Source |
|---|---|
| `trigger_message` | Orchestrator `aap_job_template` node `extra_vars` |

## Pending Work

- **External git server** — The demo was initially wired to a git endpoint running on SNO (`http://orch-demo-git.aap.svc.cluster.local:9000/cgi-bin/git/orch-demo.git`). This should be decommissioned once the demo is validated against the external GitHub repo. The SNO git service is defined in `orchestrator-demo/git-svc.yaml` in `install-nexus-operator` and should be deleted from the cluster.
- **Orchestrator webhook path** — The correct endpoint is `POST /api/v1/webhooks/demo` — there is no `/eda/` prefix despite the trigger type being `webhook_trigger`.

## Cluster Context

- OCP 4.21 SNO at `sno.stoleas.home`
- AAP 2.7: Gateway + Controller + EDA in `aap` namespace
- Automation Orchestrator EA in `automation-orchestrator-automation-orchestrator` namespace
- Orchestrator URL: `https://automa-d61a1901-automation-orchestrator-automation-orchestrator.apps.sno.stoleas.home`
- AAP Gateway URL: `https://aap-aap.apps.sno.stoleas.home`
- AAP Controller API prefix: `/api/controller/v2/` (not `/api/v2/`)
