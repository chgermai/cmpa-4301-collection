---
title: "AI in Network Operations: A Field Guide"
tags: [field-guide]
---
## Who This Is For and How to Use It

This field guide is for network engineering and operations teams that own routing, switching, and monitoring for 24/7 production systems. This guide is for you if you have tried a few AI tools but have not worked them into your day-to-day workflow. My goal is to help you filter alert noise down, allowing teams to focus on the root issues, reduce the toll of an on-call rotation, and use AI to reduce human errors.

The guide follows the life of an incident. We start with the alert, then the diagnosis, the fix, and finally the lessons learned. The goal was to allow each section to stand on its own so you are not required to read it from start to finish. Determine what stage is causing your team the most pain and start there.

To help you on the journey, I included an incident example that runs through the entire guide. Look for the examples at the end of each section to help define its context.

Every source in this guide comes from my curated collection, [[index|AI Tools for Network Engineering Teams]]. Source names link to their full annotations in the collection, and the original links are listed at the end.

Throughout the guide I used several terms:
- AIOps: using AI and machine learning to support IT operations work like alert correlation and root cause analysis.
- MTTR (mean time to resolution): the time from an alert to the issue being fixed.
- MTTK (mean time to know): the time from an alert to understanding the root cause.
- MCP (Model Context Protocol): an open standard that lets an AI assistant connect to outside tools and data, such as network devices.

## 1. Why the Operation of the Network Feels Broken

Most network teams are not short on data. They are missing the correlation. When there is a major incident, monitoring tools alert on everything, and the on-call engineer is left to sort it out. Although AI should help with the correlation, the results have been mixed.

The [[alert-noise-reduction/logicmonitor-catchpoint-sre-report|2026 SRE Report]] from LogicMonitor provides some useful information in this case, as it was based on a survey of users. One of the primary points in the report is that AI's impact has been inconsistent. The paper [[ai-assisted-troubleshooting/malik-reactive-to-autonomous|"From Reactive to Autonomous"]] helps explain why. It lays out five "generations" of network operations and argues that moving from manual to AIOps takes changes in tooling, team structure, and culture, not just a better AI model.

Buying an AI tool will not fix a process problem on its own. Before you decide on a tool, figure out where your team sits today. If alerts still go straight to a pager with no grouping, start with noise reduction before looking at AI troubleshooting. 

> [!example] Example incident
> It is early in the morning and an optic on a core uplink between the data center and the WAN edge starts failing. The interface flaps every few minutes. Each flap causes the BGP session to the WAN provider to drop, traffic shifts to the backup path and back, and application health checks fail along the way. Within ten minutes, the monitoring system has sent over a hundred alerts, including interface down, BGP neighbor down, latency, packet loss, and application checks. The on-call engineer wakes up to a phone full of pages and has to figure out which one matters.

## 2. Reducing the Noise Before It Hits You

An alert is the first stage of an incident, and it is where most of the time is wasted. The core goal of any tool used to reduce noise is the same: take otherwise disjointed alerts and reduce them to a single incident. The main difference between tools is who does the integration work.

On the enterprise side, [[alert-noise-reduction/selector-ai-network-architects|Selector AI]] correlates data across more than 300 integrations and claims to remove duplicate alerts, thereby reducing mean time to resolution. On the open-source side, [[alert-noise-reduction/keephq-keep|Keep]] centralizes alerts from over 100 tools, handling both deduplication and correlation, with automation workflows written in YAML. Keep is a self-hosted application, so your team owns the setup and maintenance. Selector, by comparison, handles most of that for you. Because of the SRE Report's finding that results vary from team to team, you should be cautious about treating those claims as expectations.

A practical approach is to test and validate both. Stand up Keep in a lab with a copy of your own alert history to learn how correlation behaves in your environment. For Selector, ask for a trial against that same data. This gives you an apples-to-apples comparison you can rely on.

> [!example] In the example
> With correlation in place, the alerts are consolidated into a single incident tied to the WAN edge router. The engineer gets one page instead of hundreds, and that page already points at the right device.

## 3. Reducing the Mean Time to Know (MTTK)

Mean time to know (MTTK) is the time between an alert being triggered and the determination of the problem. For most incidents, this is the longest stage. It is also the hardest to automate.

A [[ai-assisted-troubleshooting/cisco-community-reimagining-troubleshooting|Cisco Community discussion]] on reimagining troubleshooting makes this point succinctly: troubleshooting relies on judgment and decision-making. You can't runbook all issues. That is why it has been harder to automate troubleshooting than other tasks like software upgrades. The thread proposes running several AI agents on the same problem at once. The [[ai-assisted-troubleshooting/jillesca-ai-network-troubleshooting-poc|Jillesca proof of concept]] shows what that could look like in practice. An LLM receives a Grafana alert, connects to the network device, works out the likely root cause, and suggests a fix.

These sources come at the problem from different angles. The discussion thread is skeptical about how far automation can go, while the proof of concept shows the diagnosis step working end to end in a lab. There is agreement on one point: the AI makes a suggestion and the engineer makes the determination. This is likely the right place to start. Let the AI agent collect the show commands and give a first-pass diagnosis, and keep the decision with the engineer. 

> [!example] In the example
> An AI agent picks up the correlated incident, connects to the edge router, and pulls the interface counters, logs, and BGP summary. It reports that CRC errors on the uplink have been climbing for the past hour and the optic's receive power is below threshold. It points to a failing optic or dirty fiber and recommends shutting the interface so traffic stays on the backup path. The engineer reviews the output, agrees, and shuts down the port. This reduces the troubleshooting to the few minutes it takes to review the data and implement the fix.

## 4. Making Sure the Incident Doesn't Repeat

Once an incident is fixed, the lessons learned should turn into something that prevents it from happening again.

For known problems with known fixes, [[automation-of-repetitive-tasks/redhat-event-driven-ansible|Red Hat Event-Driven Ansible]] uses rulebooks that listen for events from monitoring tools and automatically trigger existing Ansible playbooks. This is rule-based automation, not AI. It is only as good as the rulebooks and the event sources behind it, but for teams already using Ansible it is the most direct way to stop fixing the same issue by hand.

For changes, it is a different risk. Manual changes pushed to production are one of the most common sources of incidents. The [[automation-of-repetitive-tasks/pamosima-network-mcp-docker-suite|Pamosima network MCP suite]] packages network tools as Model Context Protocol (MCP) servers in Docker, providing an AI assistant a method to query and troubleshoot the network. That provides a path to having an AI agent check a proposed change against the current state of the network before it is pushed. As with diagnosis, the AI runs the checks and the engineer approves the change.

What does a pre-change check look like? Before a change is pushed, an AI agent could do the following:
- Compare the proposed configuration to the running configuration and flag anything unexpected in the diff.
- Confirm that redundant paths and routing neighbors are healthy before work starts on the primary.
- Check for open incidents or other changes scheduled on the same devices.
- Summarize what it found in plain language so the engineer can approve or stop the change.

None of these checks are new. Engineers already do them manually. The value is that the AI runs them every time and isn't going to overlook or skip something because it is considered routine.

> [!example] In the example
> The next morning, the team schedules a change to replace the optic and bring the uplink back. Before the engineer re-enables the interface, an AI pre-check confirms the backup path and its BGP session are healthy, the new optic's light levels are in range, and no other work is scheduled on the router. The engineer approves the change and it is pushed out. As the lesson learned, the team writes an Event-Driven Ansible rulebook that states when CRC errors start climbing on a core uplink, gather the interface diagnostics and open a ticket with the data attached.

## 5. Getting Deeper into the Weeds

AI tooling for network operations sits on top of existing automation. If your team is not already using Ansible and Git, you should start there. The [[automation-of-repetitive-tasks/educative-automate-your-network|Automate Your Network]] course from Educative covers Ansible, Git, Linux, and CI/CD through hands-on exercises. It is not an AI-focused course, but it provides the foundation for the tools AI depends on. To keep up with how other teams are adopting AI, the Packet Pushers [[automation-of-repetitive-tasks/packet-pushers-network-automation-nerds|Network Automation Nerds]] podcast interviews practitioners about real deployments. Some episodes are promotional, so keep that in mind.

## Where to Start

Pick the stage causing your team the most pain.

- Too much noise
	- Set up Keep in a lab against your own alert data, then trial a vendor like Selector against the same data.
- Diagnosis takes too long
	- Try the Jillesca proof of concept in a lab.
- The same incidents keep repeating
	- Use Event-Driven Ansible rulebooks for your top recurring fixes.
- Change-induced incidents
	- Explore AI-based pre-change checks with an engineer in the loop.

You do not need to do all four at once. Pick one, prove it in a lab, and build from there.

## Sources

Chou, E. (Host). (n.d.). Network automation nerds [Audio podcast]. Packet Pushers. [https://packetpushers.net/podcast/network-automation-nerds/](https://packetpushers.net/podcast/network-automation-nerds/)

Educative. (n.d.). Automate your network [Online course]. [https://www.educative.io/courses/automate-your-network/m73KD14ZZ1A](https://www.educative.io/courses/automate-your-network/m73KD14ZZ1A)

Jantich. (n.d.). Reimagining network troubleshooting [Discussion thread]. Cisco Community. [https://community.cisco.com/t5/nso-developer-hub-discussions/reimagining-network-troubleshooting/m-p/5342470/highlight/true](https://community.cisco.com/t5/nso-developer-hub-discussions/reimagining-network-troubleshooting/m-p/5342470/highlight/true)

Jillesca. (2024). AI-Network-Troubleshooting-PoC [GitHub repository]. [https://github.com/jillesca/AI-Network-Troubleshooting-PoC](https://github.com/jillesca/AI-Network-Troubleshooting-PoC)

Keep. (n.d.). Keep [GitHub repository]. [https://github.com/keephq/keep](https://github.com/keephq/keep)

LogicMonitor. (2026). 2026 Catchpoint SRE report. [https://www.logicmonitor.com/resources/sre-report-2026](https://www.logicmonitor.com/resources/sre-report-2026)

Malik, A. (2026). From reactive to autonomous: Evolution of AI operations in cloud network infrastructure. arXiv. [https://arxiv.org/abs/2608.14574](https://arxiv.org/abs/2608.14574)

Pamosima. (n.d.). Network MCP Docker suite [GitHub repository]. [https://github.com/pamosima/network-mcp-docker-suite](https://github.com/pamosima/network-mcp-docker-suite)

Red Hat. (n.d.). Event-Driven Ansible. [https://www.redhat.com/en/technologies/management/ansible/event-driven-ansible](https://www.redhat.com/en/technologies/management/ansible/event-driven-ansible)

Selector AI. (n.d.). Network architects. [https://www.selector.ai/network-architects/](https://www.selector.ai/network-architects/)
