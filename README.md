# Awesome Agent Skills Security [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

A curated list of tools, security skills, sandboxes, benchmarks, research, and
operational practices for using third-party AI agent skills safely.

Agent skills are not just Markdown. A skill may combine instructions, scripts,
dependencies, reference files, installers, tool permissions, and remote services.
Treat it as both an untrusted prompt and an untrusted software dependency.

> [!CAUTION]
> A clean scan is evidence, not proof of safety. Skill scanners disagree, semantic
> attacks can evade static rules, and dormant behavior may only appear at runtime.
> Use layered controls and review the evidence yourself.

Last reviewed: **2026-08-23**.

## Contents

- [Threat Model](#threat-model)
- [Minimum Safe Workflow](#minimum-safe-workflow)
- [Skill Security Scanners](#skill-security-scanners)
- [Skills That Audit Skills](#skills-that-audit-skills)
- [Runtime Isolation and Policy Enforcement](#runtime-isolation-and-policy-enforcement)
- [Registries and Security Indexes](#registries-and-security-indexes)
- [Supply Chain Controls](#supply-chain-controls)
- [Benchmarks and Datasets](#benchmarks-and-datasets)
- [Standards and Security Guidance](#standards-and-security-guidance)
- [Research and Incident Analysis](#research-and-incident-analysis)
- [Community Discussions](#community-discussions)

## Threat Model

| Risk                     | Typical example                                                                       | Useful controls                                                             |
| ------------------------ | ------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Instruction injection    | A `SKILL.md` overrides higher-priority instructions, hides actions, or poisons memory | Treat skill text as untrusted data, semantic scanning, context isolation    |
| Malicious code           | Bundled scripts download payloads, execute shell commands, or establish persistence   | Static analysis, malware scanning, sandboxed detonation, read-only mounts   |
| Data exfiltration        | A skill combines credential or source-code reads with network egress                  | Capability analysis, default-deny network, secret brokering, egress logs    |
| Excessive agency         | The declared tools or actual behavior exceed the skill's stated purpose               | Least privilege, tool allowlists, human approval for consequential actions  |
| Supply chain compromise  | Typosquatting, mutable Git branches, dependency takeover, or a malicious update       | Trusted origins, commit pinning, hashes/signatures, SBOM, update diffs      |
| Composition risk         | One skill can read secrets while another can send data externally                     | Scan the installed set, analyze capability chains, isolate skills by task   |
| Path and package attacks | Symlink escape, archive traversal, hidden files, or installer metadata abuse          | Safe extraction, realpath checks, file inventory, no install-time execution |
| Scanner evasion          | Obfuscation, delayed triggers, environment checks, or behavior split across files     | Multiple engines, cross-file analysis, dynamic probes, canary credentials   |

## Minimum Safe Workflow

<!--lint disable awesome-list-item-->

1. **Quarantine before loading.** Fetch a candidate skill without adding it to an
   agent's discovery directory or privileged context. Project-level skills from a
   freshly cloned repository are untrusted too.
2. **Record provenance.** Capture the source URL, author, exact commit, tree hash,
   license, and expected purpose. Prefer an internal allowlisted mirror over live
   installation from a mutable branch.
3. **Inspect the whole bundle.** Review `SKILL.md`, scripts, references, assets,
   hooks, manifests, lockfiles, symlinks, hidden files, and every install command.
4. **Use complementary scanners.** Combine deterministic rules and code analysis
   with semantic review. Scan the complete directory and the installed skill set,
   not only the main Markdown file.
5. **Check ordinary software risk.** Scan bundled code for secrets,
   vulnerabilities, unsafe dependencies, and malware; produce an SBOM when the
   bundle has a meaningful dependency graph.
6. **Test in a disposable sandbox.** Start with no secrets, no network, a read-only
   copy of the skill, synthetic data, resource limits, and canary credentials.
   Grant only the filesystem paths, domains, tools, and environment variables the
   stated task requires.
7. **Enforce at runtime.** Put policy outside the model. Log tool calls and network
   activity, broker short-lived credentials, and require human approval for
   irreversible or cross-boundary actions.
8. **Re-verify every update.** Review the diff, rescan, update the approved hash,
   and keep an inventory so a compromised skill can be located and revoked.

<!--lint enable awesome-list-item-->

## Skill Security Scanners

Purpose-built scanners for Agent Skills, `SKILL.md`, and adjacent agent artifacts.
Review each scanner's threat model and test corpus before using its score as a gate.

- [Bawbel Scanner](https://github.com/bawbel/scanner) - Static scanner that never
  executes the target; adds OWASP AIVSS scoring, toxic-flow and delegation-chain
  checks, SARIF, accepted-risk expiry, and hash pins for drift detection.
  `Apache-2.0`.
- [Cisco AI Defense Skill Scanner](https://github.com/cisco-ai-defense/skill-scanner)
  - Best-effort scanner for prompt injection, data exfiltration, and malicious
    code patterns, with standard and lenient skill-layout support. `Apache-2.0`.
- [NVIDIA SkillSpector](https://github.com/NVIDIA/SkillSpector) - Rules across
  instruction, code, supply-chain, tool, and MCP risks; includes AST analysis,
  taint tracking, YARA signatures, and optional semantic analysis. `Apache-2.0`.
- [Snyk Agent Scan](https://github.com/snyk/agent-scan) - Discovers installed agent
  components and scans Agent Skills and MCP servers for prompt injection, secret
  handling, malicious payloads, and related risks. `Apache-2.0`.
- [SkillGate](https://github.com/charliechenye/SkillGate) - Pre-install and
  pre-merge trust checks for skills and MCP configuration, including capability
  diffs and CI policy gates. `MIT`.
- [SkillGuard](https://github.com/RudrenduPaul/skillguard) - CLI, library, MCP
  server, and GitHub Action backed by shared rule packs; includes cross-skill
  privilege-chain analysis. `Apache-2.0`.
- [SkillPreflight](https://github.com/agent-contracts/skill-preflight) - Early-stage
  static pre-install scanner for `SKILL.md` bundles used by Claude Code, Codex,
  Cursor, and Gemini CLI; source findings, JSON/SARIF output, and score, severity,
  and instruction-token CLI/CI gates. Does not execute target scripts or enforce
  runtime actions. Local scans do not upload target content; GitHub URL scans
  retrieve repository files. Heuristic coverage is limited; findings require
  manual review. `MIT`.
- [SkillScan](https://github.com/NMitchem/SkillScan) - Combines static analysis,
  LLM behavioral prediction, and optional Docker Sandbox execution with honeypot
  canaries. `MIT`.
- [skill-audit](https://github.com/ondrej-merkun/skill-audit) - Local deterministic
  scanning by default, optional local-LLM review, machine-wide discovery, GitHub
  Action integration, and an explicit threat model. `Apache-2.0`.
- [skill-veil](https://github.com/seifreed/skill-veil) - Static analysis and policy
  engine for skills, prompts, MCP manifests, and agent-adjacent artifacts, with
  JSON/SARIF output and optional multi-model false-positive review. `MIT`.
- [skilldoctor](https://github.com/studiomeyer-io/skilldoctor) - Linter and security
  scanner for `SKILL.md`, `AGENTS.md`, and subagent files; supports grading,
  auto-fixes, SARIF, and GitHub Actions. `MIT`.

### Source-Available

- [SkillFortify](https://github.com/qualixar/skillfortify) - Static analysis,
  capability verification, dependency resolution, SBOM generation, and formalized
  skill supply-chain analysis across multiple agent frameworks. `Elastic-2.0`.

## Skills That Audit Skills

These are themselves Agent Skills or skill packs. Audit and pin the auditor before
letting it inspect anything else.

- [AgentAudit Skill](https://github.com/agentaudit-dev/agentaudit-skill) - Cross-file
  review, anti-audit manipulation checks, provenance enrichment, and an
  optional shared findings registry. Review its network/reporting behavior before
  use. `AGPL-3.0`.
- [Agent Skill Security Review](https://github.com/sgolovine/agent-skills/tree/main/skills/agent-skill-security-review)
  - An adversarial review workflow for prompt injection, secret access,
    exfiltration, persistence, dependency risk, and sandbox escape. `Unlicense`.
- [Agent Security Skillpack](https://github.com/mattpartida/agent-security) - Skills
  for prompt-injection audits, configuration risk scoring, and operational
  security checklists. `MIT`.
- [Superagent Skill Security](https://github.com/superagent-ai/skills/tree/main/skills/skill-security)
  - A security-auditing skill that combines deterministic regex, AST, taint, and
    YARA checks with intent and behavior reasoning. `MIT`.

### Safely Using a Skill to Audit Another Skill

- Run the auditor in a separate, low-privilege agent or isolated context.
- Mount or quote the target as **data**, never as active trusted instructions.
- Do not execute the target's scripts or installation steps during static review.
- Remove credentials, disable network egress, and use a disposable workspace.
- Confirm important findings with a deterministic tool and a human-readable diff.
- Assume the target may try to manipulate the auditor or suppress its report.

## Runtime Isolation and Policy Enforcement

- [Anthropic Sandbox Runtime](https://github.com/anthropic-experimental/sandbox-runtime)
  - Lightweight OS-level filesystem and network restrictions for arbitrary
    processes without requiring a container. `Apache-2.0`.
- [Cloudflare Agents skill runner design](https://github.com/cloudflare/agents/blob/main/design/skills.md)
  - A useful reference implementation: deterministic skill fingerprints, explicit
    tool capabilities, no network or tools by default, read-only workspace access,
    and bounded script execution. `MIT`.
- [Docker Sandboxes security model](https://docs.docker.com/ai/sandboxes/security/)
  - MicroVM isolation, default-deny network policy, host-side credential injection,
    workspace controls, and documented trust-boundary limitations for shared skills
    and host-side MCP servers.
- [Kubernetes SIGs Agent Sandbox](https://github.com/kubernetes-sigs/agent-sandbox)
  - Kubernetes APIs for isolated, stateful agent workloads, with a documented
    threat model covering tenant, node, control-plane, network, and resource
    boundaries. `Apache-2.0`.
- [mattolson/agent-sandbox](https://github.com/mattolson/agent-sandbox) - Local
  container environment with deny-by-default egress, host-side secret injection,
  audit-friendly policy, and an agent-readable but non-authoritative policy view.
  `MIT`.
- [Microsandbox](https://github.com/superradcompany/microsandbox) - Local-first
  microVM runtime for hardware-isolated code and agent execution, with networking,
  secret, and resource controls. `Apache-2.0`.
- [OpenClaw skill security controls](https://github.com/openclaw/openclaw/blob/main/docs/tools/skills.md#security)
  - Reference controls for realpath containment, safe archive handling, dangerous
    installer-code blocking, trust envelopes, secret injection, and sandboxing.
- [Prismor](https://github.com/PrismorSec/prismor) - Self-hosted runtime control
  plane that can observe, require approval for, or block risky tool calls, secret
  leaks, prompt injection, supply-chain actions, and network egress. `Apache-2.0`.
- [SkillSandbox](https://github.com/theMachineClay/skillsandbox) - Experimental
  capability-based runtime for individual skills with explicit filesystem,
  environment, egress, syscall, and resource permissions. `Apache-2.0`.

## Registries and Security Indexes

Hosted scan results are useful discovery signals, not substitutes for local review.

- [AgentAudit Registry](https://agentaudit.dev/) - Community-contributed findings,
  package identifiers, provenance metadata, and multi-agent consensus from the
  AgentAudit skill ecosystem.
- [ClawHub security signals](https://docs.openclaw.ai/skills) - Exposes VirusTotal,
  static-analysis, and skill-scanner results plus a verifiable trust envelope for
  supported OpenClaw skills.
- [Tego Skills Security Index](https://index.tego.security/) - Searchable security
  profiles for public skills, including capability, permission, and finding views,
  with a documented API.

## Supply Chain Controls

These tools do not understand malicious natural-language instructions. Use them on
the scripts, dependencies, repository, and release artifacts bundled with a skill.

- [Cosign](https://github.com/sigstore/cosign) - Sign and verify release artifacts;
  combine signatures with immutable commit or content-hash pinning. `Apache-2.0`.
- [Gitleaks](https://github.com/gitleaks/gitleaks) - Detect secrets accidentally or
  deliberately embedded in skill repositories and their history. `MIT`.
- [OpenSSF Scorecard](https://github.com/ossf/scorecard) - Assess the security
  practices of an upstream skill repository and its dependencies. `Apache-2.0`.
- [OSV-Scanner](https://github.com/google/osv-scanner) - Check supported lockfiles,
  packages, and SBOMs for known vulnerabilities. `Apache-2.0`.
- [Semgrep](https://github.com/semgrep/semgrep) - Add language-aware rules for the
  executable code shipped with skills. `LGPL-2.1`.
- [Syft](https://github.com/anchore/syft) - Generate SBOMs from skill filesystems or
  the images used to execute them. `Apache-2.0`.
- [Trivy](https://github.com/aquasecurity/trivy) - Scan repositories, filesystems,
  and images for vulnerabilities, misconfiguration, secrets, and SBOM data.
  `Apache-2.0`.

## Benchmarks and Datasets

> [!WARNING]
> Some datasets contain malicious instructions or executable samples. Keep them
> out of normal skill discovery paths and run evaluations only in isolated,
> credential-free environments.

- [MaliciousAgentSkillsBench](https://github.com/protectskills/MaliciousAgentSkillsBench)
  - Large ecosystem snapshot, behaviorally confirmed malicious subset, static
    rules, LLM-assisted triage, and sandboxed reproduction framework. `MIT`.
- [Open Agent Security Benchmark](https://github.com/opena2a-org/oasb) - Product-agnostic
  agent attack scenarios and a dedicated skill-security scoring
  engine, with MITRE ATLAS mappings. `Apache-2.0`.
- [OpenSkillRisk](https://github.com/Miaow-Lab/OpenSkillRisk) - Real-world risky
  third-party skills paired with benign tasks and isolated environments for testing
  whether agents recognize and stop unsafe behavior. `MIT`.
- [SkillFortifyBench](https://github.com/qualixar/skillfortifybench) - Reproducible
  benign/malicious corpus across skill and agent-component formats for evaluating
  supply-chain scanners. `MIT`.
- [SkillLens](https://github.com/SkillLens-AI/skilllens) - Evaluation pipeline for
  skill utility and security using static review and sandboxed adversarial tasks.
  `Apache-2.0`.
- [SkillVetBench](https://github.com/supreme-lab/SkillVetBench) - Multi-dimensional
  LLM-as-judge evaluation and benchmark material for open-source agent skills.
  `Apache-2.0`.

## Standards and Security Guidance

- [Agent Skills specification](https://agentskills.io/specification) - The portable
  format, directory layout, metadata, scripts, references, and progressive
  disclosure model this list primarily targets.
- [Adding Agent Skills support](https://github.com/agentskills/agentskills/blob/main/docs/client-implementation/adding-skills-support.mdx)
  - Client guidance that explicitly treats freshly cloned repositories and
    project-level skills as potentially untrusted.
- [Cloudflare Agent Skills Discovery RFC](https://github.com/cloudflare/agent-skills-discovery-rfc)
  - Security considerations for trusted origins, origin allowlists, explicit
    script permission, safe parsing, and discovery.
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/)
  - Broader agentic risks including tool misuse, identity and privilege abuse,
    memory poisoning, and supply-chain compromise.
- [OWASP AIVSS](https://aivss.owasp.org/) - Scoring system for agentic AI
  vulnerabilities, including agent supply-chain and dependency risk.
- [Agents Rule of Two](https://ai.meta.com/blog/practical-ai-agent-security/) - A
  compact design rule for avoiding the dangerous combination of untrusted input,
  sensitive access, and state-changing or externally communicating actions.

## Research and Incident Analysis

- [Agent Skills in the Wild](https://arxiv.org/abs/2601.10338) - Large-scale
  empirical study, vulnerability taxonomy, detection methodology, and dataset.
- [“Do Not Mention This to the User”](https://arxiv.org/abs/2602.06547) - Study of
  malicious skills in public ecosystems with a behaviorally verified subset and
  released benchmark framework.
- [Cloak and Detonate](https://arxiv.org/abs/2607.02357) - Scanner-evasion analysis
  and the case for dynamic, instrumented skill detonation.
- [OpenSkillRisk](https://arxiv.org/abs/2607.20121) - Evaluation of whether agents
  recognize and safely handle real-world risky third-party skills.
- [Runtime Skill Audit](https://arxiv.org/abs/2606.11671) - Targeted runtime probes
  for detecting behavior that static inspection misses.
- [Snyk ToxicSkills](https://snyk.io/blog/toxicskills-malicious-ai-agent-skills-clawhub/)
  - Threat taxonomy and ecosystem study covering prompt injection, malicious
    payloads, exfiltration, persistence, and other skill supply-chain risks.
- [Towards Secure Agent Skills](https://arxiv.org/abs/2604.02837) - Lifecycle model,
  threat taxonomy, confirmed incidents, and defense directions.
- [Trust No Skill](https://unit42.paloaltonetworks.com/ai-agent-supply-chain-risks/)
  - Unit 42 analysis of agent-skill supply-chain risk and behavior-intent
    verification.

## Community Discussions

Community posts are included as discovery and operational-experience sources. Their
claims may not have been independently verified.

### Hacker News

- [Agent Skills – Open Security Database](https://news.ycombinator.com/item?id=47402118)
- [Cisco AI Agent Skills Security Scanner](https://news.ycombinator.com/item?id=46814707)
- [Show HN: Security Scanner for Agent Skills and MCP](https://news.ycombinator.com/item?id=47999709)
- [New prompt-injection papers: Agents Rule of Two and the attacker moves second](https://news.ycombinator.com/item?id=45794245)

### Reddit

- [Nobody checks what's inside Claude Code skills before installing them](https://www.reddit.com/r/ClaudeCode/comments/1qr3zc7/nobody_checks_whats_inside_claude_code_skills/)
- [How is anyone supposed to trust skills from ClawHub or GitHub?](https://www.reddit.com/r/openclaw/comments/1rgca1t/how_is_anyone_supposed_to_trust_skills_from/)
- [Agent skill marketplace supply-chain attack and scanner disagreement](https://www.reddit.com/r/netsec/comments/1s0dmuv/agent_skill_marketplace_supply_chain_attack_121/)
- [A suspicious skill found on skills.sh](https://www.reddit.com/r/ClaudeCode/comments/1sbwe7p/heads_up_i_found_a_sketchy_skills_on_skillssh/)

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) and open a
pull request with enough evidence to review the entry.

A proposed tool should:

- have publicly inspectable source and an explicit license to appear in an
  open-source section;
- document what it analyzes, what it executes, what data leaves the machine, and
  where enforcement occurs;
- explain its threat model, limitations, false-positive handling, and supported
  skill formats;
- link to tests, evaluation data, or reproducible examples where possible; and
- use a factual description without unverified “safe,” “first,” or “only” claims.

Please place unlicensed, proprietary, or hosted-only projects in a clearly labeled
section. Do not submit live malware, credential stealers, or direct installation
links to known-malicious skills.
