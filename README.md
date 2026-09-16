<h1 align="center">Renker — Trusted Infrastructure for Autonomous AI</h1>

<p align="center">
  <em>Deterministische Autorisierung, nachvollziehbare Audit-Trails und inhaltsblinde E2E-Kommunikation<br/>für eine Welt, in der KI-Agenten eigenständig handeln.</em>
</p>

<p align="center">
  <a href="https://sebastianrenker.github.io">🌐 Landingpage</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/License-Apache_2.0-blue?style=flat" alt="Apache-2.0"/>
  <img src="https://img.shields.io/badge/License-MIT-green?style=flat" alt="MIT"/>
</p>

---

## Worum es geht

Autonome KI-Agenten treffen Entscheidungen und lösen Aktionen aus, bevor ein Mensch
mitliest. **Renker** ist ein Forschungs- und Prototyp-Ökosystem, das die fehlende
Vertrauensschicht dafür baut: Jede Agenten-Aktion läuft durch eine deterministische
Policy-Engine (`ALLOW` / `DENY` / `REQUIRE_APPROVAL`), landet in einem manipulations­erkennenden
(tamper-evident, nicht tamper-proof) Audit-Log und kommuniziert – wo nötig – über einen
Ende-zu-Ende-verschlüsselten Kanal mit Post-Quantum-Hybrid-Handshake (X25519 + ML-KEM-768).

Das adressiert konkrete Angriffsflächen agentischer Systeme – allen voran **Prompt-Injection**,
bei der ein Agent aus manipuliertem Kontext heraus Aktionen ausführt, die der Nutzer nie
autorisiert hat.

> ⚠️ **Status:** Forschungs-Prototypen (Phase 0). Nicht für den Produktivbetrieb gedacht.

---

## Wie die Bausteine zusammenhängen

```mermaid
flowchart TB
    subgraph agents["KI-Agenten (Anwendungsschicht)"]
        rencora["<b>rencora</b><br/>Desktop-KI-Assistent<br/><i>Python · PyQt6</i>"]
        continuum["<b>continuum</b><br/>kontinuierlich lernendes<br/>Forschungssystem<br/><i>Python</i>"]
    end

    subgraph trust["Vertrauensschicht (Renker Platform)"]
        core["<b>renker-core</b><br/>Identity · Permissions<br/>Audit · Policy<br/><i>Platform-Foundation</i>"]
        authz["<b>renker-core-authz</b><br/>Capability + Policy-Engine<br/>+ tamper-evident Audit<br/><i>Apache-2.0</i>"]
    end

    subgraph comms["Sichere Kommunikation"]
        vault["<b>renkervault</b><br/>Inhaltsblinder E2E-Chat<br/>Double Ratchet · PQ-Hybrid<br/><i>TypeScript</i>"]
    end

    subgraph ops["Qualität &amp; Betrieb (querschnittlich, Dev-Zeit)"]
        custos["<b>custos</b><br/>Evidenzbasierte Verifikation<br/>von Agenten-Behauptungen<br/><i>Claude-Code-Plugin</i>"]
        flint["<b>renker-flint</b><br/>Token-Reduktions-<br/>Mess- &amp; Betriebsschicht<br/><i>Konzeptphase</i>"]
    end

    rencora -->|"Tool-Aufruf zur Prüfung"| core
    continuum -->|"Aktion zur Prüfung"| core
    core -->|"delegiert Autorisierungs-<br/>entscheidung"| authz
    authz -->|"ALLOW / DENY /<br/>REQUIRE_APPROVAL"| core
    core -.->|"schreibt Audit-Trail"| authz
    agents -.->|"verschlüsselter Kanal"| vault
    custos -.->|"CI-Gate: belegt Behauptungen<br/>mit ausgeführten Checks"| trust
    flint -.->|"misst &amp; senkt<br/>Session-Token-Kosten"| agents

    classDef agent fill:#1f2937,stroke:#4b5563,color:#f9fafb;
    classDef trustnode fill:#0f3d3e,stroke:#14b8a6,color:#f0fdfa;
    classDef commsnode fill:#3b0764,stroke:#a855f7,color:#faf5ff;
    classDef opsnode fill:#3f2d0a,stroke:#f59e0b,color:#fffbeb;
    class rencora,continuum agent;
    class core,authz trustnode;
    class vault commsnode;
    class custos,flint opsnode;
```

**Kurz gesagt:** Agenten (`rencora`, `continuum`) fragen vor jeder sicherheitsrelevanten
Aktion die Vertrauensschicht. `renker-core` bündelt Identität, Rechte, Audit und Policy als
Plattform-Foundation und delegiert die eigentliche Autorisierungsentscheidung an
`renker-core-authz` – die deterministische, quelloffene Policy-Engine. `renkervault` liefert
den verschlüsselten Kanal, wenn Agenten oder Nutzer vertraulich kommunizieren müssen.
Querschnittlich, zur Entwicklungszeit (nicht zur Laufzeit integriert): `custos` erzwingt in
CI, dass Agenten-Behauptungen mit **ausgeführten** Checks belegt werden statt nur behauptet,
und `renker-flint` misst und senkt die Token-Kosten read-lastiger Sessions – beides nach
derselben Evidenz-Regel wie der Rest der Plattform (belegen statt behaupten).

---

## Repositories

| Repo | Rolle | Stack |
|------|-------|-------|
| [**renker-core-authz**](https://github.com/sebastianrenker/renker-core-authz) | Deterministische Capability- + Policy-Engine mit tamper-evident Audit für Agenten-Aktionen | Python · Apache-2.0 |
| [**renker-core**](https://github.com/sebastianrenker/renker-core) | Gemeinsame Plattform-Foundation: Identity, Permissions, Audit, Policy | Python |
| [**custos**](https://github.com/renker-industries/custos) | Evidenzbasierte Verifikation von Agenten-Behauptungen: erzwingt in CI **ausgeführte** Checks (Linter, Tests, Statik) statt bloßer Zusagen | Python · MIT |
| [**continuum**](https://github.com/sebastianrenker/continuum) | Prototyp eines kontinuierlich lernenden KI-Forschungssystems (Memory, Bayesian Optimization, Safety Gates, Anti-Hallucination) | Python |
| [**renkervault**](https://github.com/sebastianrenker/renkervault) | Inhaltsblinder E2E-verschlüsselter Chat-Prototyp (Double Ratchet, PQ-Hybrid-Handshake, Duress-Modus) | TypeScript |
| [**rencora**](https://github.com/sebastianrenker/rencora) | Persönlicher Desktop-KI-Assistent (Sprache, Screen/Kamera, Agenten, Gedächtnis) | Python · PyQt6 |
| [**renker-flint**](https://github.com/sebastianrenker/renker-flint) | Token-Reduktions-Mess- & Betriebsschicht für read-lastige Agent-Sessions; belegt Einsparungen provider-seitig statt sie zu behaupten (Konzeptphase) | Konfig · Messung · MIT |
| [**sebastianrenker.github.io**](https://github.com/sebastianrenker/sebastianrenker.github.io) | Plattform-Landingpage & Doku-Hub | HTML/CSS |

---

## Schwerpunkte

`agentic-ai` · `authorization` · `policy-engine` · `prompt-injection` · `tamper-evident-audit`
· `end-to-end-encryption` · `post-quantum-hybrid` · `metadata-minimization` · `agent-verification`
· `evidence-driven` · `token-efficiency`

---

<p align="center"><sub>🌐 <a href="https://sebastianrenker.github.io">sebastianrenker.github.io</a></sub></p>
