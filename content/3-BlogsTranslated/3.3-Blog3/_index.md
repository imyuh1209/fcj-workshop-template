---
title: "AWS Security Agent: A single agent covering the entire application development lifecycle"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

One of the eternal problems in security is that it is often isolated from the development process: designers design, developers write code, and only then does the security team review it—usually when it is too late and expensive to fix. AWS Security Agent (part of AWS Continuum) is trying to eliminate that gap by embedding security across three stages: during design, during coding, and during deployment—all within a single agentic service.

The latest update (mid-June 2026) introduces threat modeling, expands code review to multiple Git platforms, and allows everything to run directly within the IDE. Here are the highlights.

## Threat modeling: Integrating security from the design phase

This is a new and perhaps the most notable feature. Instead of waiting for the code to run before scanning for vulnerabilities, the Security Agent directly reads design documents or source code, reconstructing the data flow, architecture, and trust boundaries of the application. From there, it identifies potential threat actors and attack vectors, prioritizing which threats need to be addressed first based on the STRIDE framework (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege).

The great thing is that it works right from the design phase—when the cost of fixing a vulnerability is still cheap, without the need to refactor already shipped code.

## Code review: Expanding platforms, deeper pattern-matching

Previously, the Security Agent could only scan on GitHub. Now, it has added support for GitLab and Bitbucket (both SaaS and self-hosted versions), along with the ability to pull documentation from Confluence for review context.

Its review methodology is not simply matching known error patterns like traditional linters. Instead, it uses reasoning to find complex vulnerabilities, checks them against the organization's specific security requirements, and crucially, validates them in a simulation environment to confirm if the vulnerability is actually exploitable. This avoids flooding developers with false positives that waste their time.

Furthermore, design reviews have been upgraded with built-in compliance packs (AWS Well-Architected Framework, NIST CSF, PCI DSS, etc.) or the ability to import custom security requirements from internal documents. Every finding maps back to the compliance posture, keeping the team audit-ready at all times.

## Running directly in the IDE without switching tabs

This solves a very real pain point: previously, developers had to leave the IDE and open a separate console to view findings or threat models. Now, with Kiro power, the Claude Code plugin (officially named: AWS Agents for DevSecOps), and MCP integration open to any AI IDE, everything runs right inside the editor using natural prompts, for example:

- `"Run a full security scan on this repo"` → scans the entire repository
- `"Build a threat model for this application"` → saves the threat model to `.security-agent/threat_model.md` in the workspace
- `"help me remediate my findings"` → the agent pulls findings, prioritizes the most severe ones, and immediately opens a bugfix session to resolve them

This approach makes sense: the security agent is not trying to replace the developer, but rather trying to "meet" the developer right where they are working.

## A few notes when testing

The upgraded threat modeling and code review features are currently in Preview, so their behavior might change. If you want to try it out, the Security Agent offers a 2-month free trial—but you should still monitor your account and credits carefully to avoid unexpected costs. Also, remember: the agent reads source code directly to build the threat model, so if your repository contains secrets or credentials, you should clean them before letting the agent scan.

## Conclusion

Overall, the direction of AWS Security Agent is to transform security from an isolated, final check into a continuous part of the workflow—from the design document, through each pull request, to deployment. For those working on projects that handle sensitive data (like uploading user photos/videos), this is a tool worth trying to better protect your system and practice systematic threat modeling thinking.

---

## References
- [AWS Security Agent adds threat modeling, Kiro power, and Claude Code plugin, and more](https://aws.amazon.com/vi/blogs/aws/aws-security-agent-adds-threat-modeling-kiro-power-and-claude-code-plugin-and-more/)
