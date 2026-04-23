# SOAR + EDR Integration and Automation Project

![Platform](https://img.shields.io/badge/Platform-Tines-blue)
![EDR](https://img.shields.io/badge/EDR-LimaCharlie-black)
![Automation](https://img.shields.io/badge/Focus-SOAR%20Automation-success)
![Lab](https://img.shields.io/badge/Environment-Windows%20Lab-informational)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Overview

This project demonstrates a hands-on SOAR and EDR integration workflow built in a controlled lab. The lab uses **LimaCharlie** for endpoint telemetry and detection, **Tines** for orchestration, and **Slack / email** for analyst notifications. To validate the workflow, I executed **LaZagne** in an isolated Windows lab environment and used that activity to trigger an automated response path.

The final workflow shows how a detection can move from endpoint telemetry to analyst decision-making and optional endpoint isolation.

> [!IMPORTANT]
> Offensive tooling was used **only inside an isolated lab** to validate defensive detections and response automation.

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Lab Stack](#lab-stack)
- [Workflow Summary](#workflow-summary)
- [Environment](#environment)
- [Walkthrough](#walkthrough)
  - [1) Playbook design](#1-playbook-design)
  - [2) LimaCharlie sensor deployment](#2-limacharlie-sensor-deployment)
  - [3) Detection engineering for LaZagne](#3-detection-engineering-for-lazagne)
  - [4) Slack preparation](#4-slack-preparation)
  - [5) Tines automation build](#5-tines-automation-build)
  - [6) End-to-end validation](#6-end-to-end-validation)
- [Project Outcomes](#project-outcomes)
- [What Employers Should Notice](#what-employers-should-notice)
- [Lessons Learned](#lessons-learned)
- [Future Improvements](#future-improvements)
- [Repository Structure](#repository-structure)

## Objectives

- Build a practical SOAR + EDR lab workflow.
- Generate a custom detection for suspicious process execution.
- Send detection data from LimaCharlie into Tines.
- Notify analysts through Slack and email.
- Add a human decision point before containment.
- Validate host isolation and report the final state back to the analyst.

## Lab Stack

- **Tines** - workflow orchestration and decision handling
- **LimaCharlie** - endpoint telemetry, detections, and response actions
- **Windows VM** - test endpoint
- **Slack** - alerting and analyst communication
- **Email** - secondary alerting path
- **LaZagne** - test utility used to simulate credential access behavior in the lab

## Workflow Summary

1. A Windows endpoint sends telemetry into LimaCharlie.
2. A custom detection rule identifies execution associated with **LaZagne**.
3. LimaCharlie forwards detection data into a **Tines webhook**.
4. Tines sends notifications to **Slack** and **email**.
5. Tines presents a **Yes / No analyst prompt** asking whether to isolate the system.
6. If **Yes**, Tines issues an isolation request and confirms the result.
7. If **No**, Tines sends a follow-up Slack message indicating manual investigation is required.

## Environment

- Windows virtual machine
- LimaCharlie tenant
- Tines story with webhook and response actions
- Slack workspace / alerts channel
- Email notification path

## Walkthrough

## 1) Playbook design

I started by diagramming the automation path so the detection, alerting, analyst decision, and containment steps were clear before building the workflow.

### Workflow diagram

![01-playbook-workflow-diagram.png](images/01-playbook-workflow-diagram.png)

## 2) LimaCharlie sensor deployment

The first operational step was preparing the Windows endpoint and enrolling it into LimaCharlie using an installation key.

### Installation keys view

![02-limacharlie-installation-keys.png](images/02-limacharlie-installation-keys.png)

### Sensor key

![03-limacharlie-sensor-key.png](images/03-limacharlie-sensor-key.png)

### PowerShell sensor installation

```powershell
.\hcp_win_x64_release_4.29.2.exe -i [Sensor Key]
```

![04-sensor-installation-powershell.png](images/04-sensor-installation-powershell.png)

## 3) Detection engineering for LaZagne

After onboarding the endpoint, I executed **LaZagne** in the lab and reviewed the resulting telemetry inside LimaCharlie. I then used that event data to build and test a custom Detection & Response rule.

### LaZagne execution on the Windows endpoint

![05-lazagne-execution-on-windows.png](images/05-lazagne-execution-on-windows.png)

### Locating the process in LimaCharlie timeline

![06-limacharlie-timeline-lazagne-process.png](images/06-limacharlie-timeline-lazagne-process.png)

### Creating a new D&R rule

![07-limacharlie-new-dr-rule.png](images/07-limacharlie-new-dr-rule.png)

### Rule editor

![08-limacharlie-rule-editor.png](images/08-limacharlie-rule-editor.png)

### Starting from a similar rule

![09-limacharlie-similar-rule-search.png](images/09-limacharlie-similar-rule-search.png)

### Reviewing a reference rule on GitHub

![10-github-rule-reference.png](images/10-github-rule-reference.png)

### Copying the raw rule content

![11-github-raw-rule-content.png](images/11-github-raw-rule-content.png)

### Pasting and editing the rule in LimaCharlie

![12-limacharlie-rule-paste-and-edit.png](images/12-limacharlie-rule-paste-and-edit.png)

### Detection logic refinement

![13-limacharlie-detect-rule-logic.png](images/13-limacharlie-detect-rule-logic.png)

### Response logic refinement

![14-limacharlie-respond-rule-logic.png](images/14-limacharlie-respond-rule-logic.png)

### Event conditions and matching fields

![15-limacharlie-rule-event-conditions.png](images/15-limacharlie-rule-event-conditions.png)

### Rule test configuration

![16-limacharlie-rule-test-configuration.png](images/16-limacharlie-rule-test-configuration.png)

### Successful test result

![17-limacharlie-rule-test-match-true.png](images/17-limacharlie-rule-test-match-true.png)

## 4) Slack preparation

Before wiring in orchestration, I prepared a Slack alerts channel so detections and response outcomes could be surfaced immediately to an analyst.

### Slack alerts channel

![18-slack-alerts-channel.png](images/18-slack-alerts-channel.png)

## 5) Tines automation build

Next, I built the orchestration flow in Tines. This included the inbound webhook, notification actions, analyst prompt, branching logic, and the HTTP request used to isolate the endpoint.

### New Tines story

![19-tines-new-story.png](images/19-tines-new-story.png)

### Webhook action

![20-tines-webhook-action.png](images/20-tines-webhook-action.png)

### Add output in LimaCharlie

![21-limacharlie-add-output.png](images/21-limacharlie-add-output.png)

### Output type: detections

![22-limacharlie-output-detections.png](images/22-limacharlie-output-detections.png)

### Output destination: Tines

![23-limacharlie-output-tines.png](images/23-limacharlie-output-tines.png)

### Webhook output configuration

![24-limacharlie-output-webhook-config.png](images/24-limacharlie-output-webhook-config.png)

### Output saved successfully

![25-limacharlie-output-saved.png](images/25-limacharlie-output-saved.png)

### Slack and email actions in Tines

![26-tines-slack-and-email-actions.png](images/26-tines-slack-and-email-actions.png)

### Analyst prompt action

![27-tines-user-prompt-action.png](images/27-tines-user-prompt-action.png)

### Yes / No trigger paths

![28-tines-yes-no-triggers.png](images/28-tines-yes-no-triggers.png)

### No-path Slack notification

![29-tines-no-trigger-slack-action.png](images/29-tines-no-trigger-slack-action.png)

### HTTP request to isolate sensor

![30-tines-isolate-sensor-http-request.png](images/30-tines-isolate-sensor-http-request.png)

### Final Tines workflow

![31-tines-final-story-workflow.png](images/31-tines-final-story-workflow.png)

## 6) End-to-end validation

Finally, I ran the workflow from beginning to end to confirm that detection, alerting, prompting, and containment all behaved as expected.

### Initial malicious-tool execution in the lab

![32-lazagne-triggered-event.png](images/32-lazagne-triggered-event.png)

### Connectivity before isolation

![33-pre-isolation-network-connectivity-test.png](images/33-pre-isolation-network-connectivity-test.png)

### Slack detection alert

![34-slack-lazagne-detection-alert.png](images/34-slack-lazagne-detection-alert.png)

### Email detection alert

![35-email-lazagne-detection-alert.png](images/35-email-lazagne-detection-alert.png)

### Analyst isolation prompt

![36-analyst-isolation-prompt.png](images/36-analyst-isolation-prompt.png)

### Analyst selected **Yes**

![37-analyst-selected-yes.png](images/37-analyst-selected-yes.png)

### Tines yes-path execution

![38-tines-yes-path-execution.png](images/38-tines-yes-path-execution.png)

### Host isolation visible in LimaCharlie

![39-limacharlie-sensor-isolated.png](images/39-limacharlie-sensor-isolated.png)

### Connectivity after isolation

![40-post-isolation-network-test.png](images/40-post-isolation-network-test.png)

### Slack confirmation of isolation

![41-slack-isolation-confirmation.png](images/41-slack-isolation-confirmation.png)

### Analyst selected **No** on a later test

![42-analyst-selected-no.png](images/42-analyst-selected-no.png)

### Tines no-path execution

![43-tines-no-path-execution.png](images/43-tines-no-path-execution.png)

### Slack message requesting investigation

![44-slack-no-isolation-investigate.png](images/44-slack-no-isolation-investigate.png)

## Project Outcomes

This lab successfully demonstrated that:

- endpoint activity can be captured and translated into a practical detection workflow,
- detection data can be forwarded into an orchestration platform,
- analyst-facing notifications can be sent automatically,
- a human approval checkpoint can be inserted before containment,
- and endpoint isolation status can be confirmed and reported back to stakeholders.

## What Employers Should Notice

- **Detection engineering:** I reviewed raw event data and turned it into a targeted D&R rule.
- **Security automation thinking:** I designed a workflow that balances automated action with analyst approval.
- **Tool integration:** I connected endpoint detection, orchestration, and communications tooling into one process.
- **Validation mindset:** I tested both the **Yes** and **No** containment paths instead of stopping at a single happy-path demo.
- **Documentation:** I organized the project so another reviewer can quickly understand the workflow, decision points, and outcomes.

## Lessons Learned

- Strong workflow design up front makes automation troubleshooting much easier later.
- Detection tuning matters because useful automation depends on reliable inputs.
- Human decision points can add safety when response actions affect endpoint availability.
- Validation should include both technical success criteria and analyst communication outcomes.

## Future Improvements

- Add a sanitized sample of the detection logic used in LimaCharlie.
- Add a redacted example payload from the webhook into Tines.
- Expand the response path to create a case or ticket automatically.
- Add recovery / unisolation steps and post-incident documentation workflows.
- Map the detection to MITRE ATT&CK techniques with a short rationale section.

## Repository Structure

```text
SOAR-EDR-Integration-and-Automation-Project/
├── README.md
└── images/
    ├── 01-playbook-workflow-diagram.png
    ├── 02-limacharlie-installation-keys.png
    ├── 03-limacharlie-sensor-key.png
    ├── ...
    └── 44-slack-no-isolation-investigate.png
```

## Notes

- Keep screenshots inside the local `images/` folder instead of hotlinking from Imgur.
- Use descriptive filenames so the walkthrough is easier to maintain.
- If any screenshot contains sensitive values, redact them before pushing to GitHub.
