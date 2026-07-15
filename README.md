![OpenASC Logo](git-assets/openasc-logo.svg)

# OpenASC-Catalogs

OpenASC is an open-source repository of structured threat, control, and asset catalogs. It provides a foundational resource for systematically identifying, assessing, and mitigating security threats in the automotive domain utilizing classification models like STRIDE and MITRE ATT&CK®. OpenASC is designed to actively support the Risk Assessment and TARA processes outlined in Clause 15 of **ISO/SAE 21434** and the threat/mitigation requirements of **UN Regulation No. 155**. Select controls (Secure Firmware Update, Anti-Rollback Protection, Crypto-Agility) also support the software update management practices required by **UN Regulation No. 156**, though full R156 compliance requires a broader software update management system (SUMS) this catalog does not itself provide.

## Features
- **Threat Catalog**: A comprehensive, interconnected list of automotive-specific threats, classified via STRIDE/MITRE ATT&CK® and mapped to UN R155 Annex 5 threat categories.
- **Control Catalog**: Standardized security controls (cryptographic, hardware, process) mathematically designed to reduce attack feasibility, mapped to UN R155 mitigations.
- **Asset Catalog**: A detailed physical and logical inventory of the vehicle ecosystem, following different E/E architectures.
- **Graph-Ready Capability Taxonomy**: Utilizes a strict, Mutually Exclusive and Collectively Exhaustive (MECE) capability syntax (`Action:Class:Domain`) to map dynamic attack paths using `requires`, `grants` (Threats) and `denies`, `overriddenBy` (Controls) for logic gates.
- **AI Vector Integration**: Schema is designed for seamless integration with locally-executed LLM embedding engines for NLP-powered threat relevance scoring.
- **JSON Schema Validation**: All catalogs are validated against strict JSON Schemas to ensure logical consistency and programmatic interoperability.

## Project Structure
```text
catalogs/         # The main catalogs for assets, threats, and controls
schemas/          # JSON Schemas for assets, threats, and controls
scripts/          # Validation scripts (Node.js)
LICENSE.md        # MIT License
package.json      # Project metadata and scripts
README.md         # Project documentation
CONTRIBUTING.md   # Project contribution guide
```

## Getting Started

### Prerequisites

* Node.js (v16 or later recommended)

### Install Dependencies

```sh
npm install
```

### Validate Catalogs

```sh
# Validate threats catalog
npm run validate:threats

# Validate controls catalog
npm run validate:controls

# Validate assets catalog
npm run validate:assets

# Validate all catalogs
npm run validate:all
```

## Catalog Formats

* **Assets**: Defined in `catalogs/assets.json`, validated by `schemas/asset.schema.json`.
* **Threats**: Defined in `catalogs/threats.json`, validated by `schemas/threat.schema.json`.
* **Controls**: Defined in `catalogs/controls.json`, validated by `schemas/control.schema.json`.

---

## The OpenASC Asset & Impact Methodology

To align with ISO/SAE 21434, OpenASC natively separates damage impacts into two distinct stakeholder perspectives: `oem` and `roadUser`, with the capability of adding `supplier`.

**`oem.safety` is structurally almost always `None`.** ISO/SAE 21434's TARA is road-user-centric - an OEM as a corporate entity has no direct physical safety state that a compromise can affect. Whatever safety-adjacent exposure an OEM carries from a safety-relevant compromise is a *consequence* of road-user harm, expressed monetarily (recall costs, liability judgments, regulatory fines) - that belongs in `oem.financial`/`oem.operational`, not a parallel `oem.safety` scale. Rate `oem.safety` as `None` with a one-line rationale pointing to where the real exposure lives, and reserve any non-`None` value for the rare case where the OEM genuinely bears a direct (not consequential) physical safety exposure.

---

## Automotive Threat Modeling Taxonomy (Structure vs. Payload)

To support automated threat modeling and graph-based attack path analysis without combinatorial explosion, OpenASC utilizes a strict, highly condensed capability syntax formatted as: **`Action:Class:Domain`**.

This taxonomy abstracts away hyper-specific technologies into 8 core functional classes, allowing engineers to model legacy E/E architectures, modern Software-Defined Vehicles (SDVs), Advanced Hardware Fault Injection, and Post-Quantum Cryptography (PQC) threats using a single, unified mathematical logic.

This is known internally as the **$8 \times 8 \times 8 \times 88$ Matrix** - the metadata dictionary is a curated 88-tag payload (see §4).

* **The Structure ($8 \times 8 \times 8$):** A strictly controlled syntax formatted as **`Action:Class:Domain`**. This mathematically guarantees that graph routing algorithms resolve without infinite loops, allowing legacy ICE architectures and modern SDVs to be modeled with the exact same math. `Domain` additionally accepts a reserved wildcard (`*`) for platform-level capabilities that are not bound to a single domain - see §3 below for matching semantics.
* **The Payload (88 Tags):** An array of curated Engineering Metadata Tags (`metadata: string[]`). This brings the graph back to engineering reality by specifying the exact protocol, hardware, or mechanism involved (e.g., separating a generic "Network Inject" into specific `[CAN-FD]` vs `[Automotive Ethernet]` vectors).

### 1. ALLOWED ACTIONS (The Verbs)

The action list relies on exactly 8 mutually exclusive attacker operations. This forces threat models to describe the *mechanism* of the attack.

* **`Extract`**: Read data or code at rest (e.g., memory dumping, extracting a key, SCA).
* **`Intercept`**: Read data or code in transit (e.g., passively sniffing a bus, wiretapping).
* **`Modify`**: Alter data or code at rest (e.g., patching a binary, changing a config).
* **`Inject`**: Introduce malicious data or commands in transit (e.g., spoofing messages, API injection).
* **`Disrupt`**: Exhaust resources or deny service (e.g., flooding a network, cutting a physical wire).
* **`Execute`**: Run arbitrary code or commands (e.g., remote code execution).
* **`Elevate`**: Escalate privileges or logical state (e.g., bypassing a secure boot check, User to Root).
* **`Fault`**: Physically manipulate hardware state (e.g., voltage glitching, electromagnetic fault injection, DFA).

### 2. ALLOWED ASSET CLASSES (The Nouns)

* **`Compute`**: Physical processing units (e.g., MCU, SoC, HSM, GPU, FPGA).
* **`Memory`**: Physical storage hardware (e.g., Flash, RAM, EEPROM, eFuse).
* **`Peripheral`**: Edge hardware components (e.g., Sensors, Actuators, Transceivers).
* **`Network`**: Logical communication channels and protocol stacks, regardless of whether they operate within the vehicle or across external trust boundaries (e.g., CAN, Automotive Ethernet, Cellular link, TLS session, REST API, V2X channel).
* **`Interface`**: Physical hardware access points - the connector, port, pad, or antenna feed where a channel is established or directly probed (e.g., OBD-II port, USB connector, JTAG pads, EVSE plug).
* **`Executable`**: Compiled software, logic, or active instructions (e.g., Bootloader, OS, Firmware, App).
* **`Data`**: Passive information at rest (e.g., Configurations, Logs, PII, Calibration data, ML model weights, FPGA bitstreams).
* **`Crypto`**: Cryptographic security material (e.g., Keys, Certificates, Seeds, Entropy).

### 3. ALLOWED DOMAINS (The Trust Boundaries)

* **`Powertrain`**: Propulsion, energy, and battery management.
* **`Chassis`**: Steering, braking, and suspension.
* **`Body`**: Lighting, access, and comfort systems.
* **`Infotainment`**: User interfaces, displays, and cabin OS.
* **`ADAS`**: Autonomous driving, perception, and decision algorithms.
* **`Telematics`**: External connectivity and gateway routing.
* **`Backend`**: OEM or third-party cloud infrastructure and services.
* **`External`**: Off-board entities (e.g., Mobile Apps, EVSE, Diagnostic Tools, Key Fobs).
* **`*` (wildcard)**: Reserved for capabilities and assets that are not bound to a single domain - platform-level primitives such as a root of trust, a hypervisor, or a shared secure element that underpin every domain rather than one. This brings the `domain` enum to 9 values in total; 8 are named vehicle domains, and the wildcard is a routing sentinel, not a 9th domain - the "8×8×8" structure is unchanged.

**Wildcard matching in the capability graph:** when the graph engine checks whether a held capability (from a threat's `grants`) satisfies a required capability (in another threat's `requires`, or a control's `denies`/`overriddenBy`), the `domain` field matches if either side is `*`, in addition to an exact match. `action` and `assetClass` never wildcard - they always require an exact match. This keeps the matcher O(1) per comparison and prevents a single capability from silently satisfying every action/class combination in a domain. Where a threat or control genuinely spans several specific actions or classes, use an explicit `OR` condition (`requires`/`overriddenBy`) or list multiple entries (`grants`/`denies`) instead of relying on a wildcard.

**Scoped resolution - `*` is a template parameter, not a global joker.** A large share of the catalog's threats are generic technique templates (e.g. runtime memory tampering, privilege escalation via a compromised executable) whose mechanism genuinely doesn't depend on which vehicle domain hosts it - that's why their `requires`/`grants` use `*`. This does **not** mean a capability granted in one real domain should satisfy a prerequisite in an unrelated one. A consuming engine must resolve `*` **per analysis, scoped to the target domain being studied** - when building an attack tree or feasibility path that terminates in, say, Infotainment, every `*` node touched by that traversal (on both the `requires` and `grants` side) binds to `Infotainment` for the duration of that analysis, not to whichever domain happened to appear elsewhere in the graph. Concretely: a threat with `grants: [{action:"Modify", assetClass:"Memory", domain:"*"}]`, instantiated against an Infotainment-scoped analysis, grants `Modify:Memory:Infotainment` - never `Modify:Memory:Backend` or any other domain, even though the catalog entry itself is written generically. This is what keeps the same technique definition reusable across every domain without duplicating it 8 times, while preserving real domain-boundary behavior in the resulting attack graph.

### 4. ALLOWED METADATA (The Engineering Payload)

When defining an Asset, Threat, or Control in the JSON catalogs, engineers must apply the relevant metadata from this 88-item curated dictionary to ensure AI and UI filters resolve accurately.

* **1. In-Vehicle Networking (IVN)**: `CAN / CAN-FD / CAN-XL`, `Automotive Ethernet`, `LIN`, `FlexRay`, `SerDes`, `PCIe`, `SPI / I2C / I3C`, `UART / USART`, `CXPI`, `SENT / PSI5`, `TSN / AVB (Time-Sensitive Networking)`
* **2. Wireless & External RF**: `Cellular`, `Wi-Fi`, `Bluetooth / BLE`, `UWB`, `NFC / RFID`, `GNSS`, `LF / RF`, `V2X`, `Satellite`, `mmWave`, `RKE / PEPS (Keyless Entry)`
* **3. Data & App Protocols**: `UDS`, `DoIP`, `SOME/IP`, `DDS`, `MQTT / AMQP`, `XCP / CCP`, `J1939 / FMS`, `ISO 15118 / OCPP`, `gRPC / REST`, `LWM2M / OMA-DM`, `OTA / SOTA / FOTA`
* **4. Interfaces & Diagnostics**: `OBD-II`, `JTAG / cJTAG`, `SWD`, `USB`, `HDMI / DisplayPort`, `Audio Jacks / A2B`, `EVSE Plug / Wireless Charging`, `Trailer Socket`, `Media Slots`, `Diagnostic Pads`
* **5. Hardware & Silicon**: `MCU`, `MPU`, `SoC`, `HPC / Zonal Controller`, `GPU / NPU`, `Sensor / Actuator`, `Transceiver / PHY`, `Flash / RAM / EEPROM`, `FPGA`, `eFuse / OTP`, `LiDAR / Radar / Camera`
* **6. Software Environments**: `RTOS`, `HLOS`, `Hypervisor`, `Container`, `Bootloader`, `Middleware`, `App / Service`, `Kernel / Driver`, `ML Model / Weights`, `Config / Calibration File`, `SBOM / Software Composition`, `LLM / Conversational AI`
* **7. Security Mechanisms**: `HSM / SHE`, `TEE`, `TPM / SE`, `SecOC`, `MACsec / IPsec`, `TLS / DTLS`, `Secure Boot`, `TRNG / PRNG`, `PUF`, `IDS / IPS`, `OAuth / OIDC / SAML`
* **8. Cryptographic Assets**: `PKI / Certificates`, `Symmetric Keys`, `Asymmetric Keys`, `Hashes`, `MAC / HMAC / CMAC`, `Digital Signatures`, `Nonces / IVs`, `Passwords / PINs`, `Tokens / API Keys`, `Seeds / RoT Data`, `Session Keys / Ephemeral Secrets`

**A curation tradeoff, stated plainly:** roughly half of these 88 tags bundle 2-3 related technologies into one string (`CAN / CAN-FD / CAN-XL`, `TLS / DTLS`, `MAC / HMAC / CMAC`, `EVSE Plug / Wireless Charging`, etc.) rather than splitting each into its own selectable tag. This keeps the dictionary a curated 88 items instead of exploding past 150, but it means filtering/tagging precision stops at the bundle, not the individual protocol variant - an entry tagged `CAN / CAN-FD / CAN-XL` can't be narrowed to "CAN-FD only" without reading its `description`. If your use case needs finer-grained filtering than a bundle provides, that detail belongs in the entry's `description`, not a new tag. Group 6 (Software Environments) holds 12 tags and Group 4 (Interfaces & Diagnostics) holds 10, rather than 11 each - `LLM / Conversational AI` was appended to Group 6 and `Wireless Charging Interface` was folded into `EVSE Plug` in Group 4, both on their individual merits rather than to preserve an "8 groups of 11" symmetry that was never a load-bearing constraint.

---

## Graph-Ready Schema & Capability Model

The OpenASC catalog schema is designed to be consumed by the attack graph engines present in [openasc.org](https://openasc.org). The fields below define logical capability dependencies across catalog entries - understanding their semantics is essential for contributing accurate entries.

### Capability Fields

**On Threat entries:**

* **`requires`**: A logical condition (`AND` / `OR` tree of `Action:Class:Domain` nodes) expressing what attacker capability must already be present before this threat can be executed. `AND` means the attacker needs all listed capabilities; `OR` means any one is sufficient. Single-node conditions may be expressed directly without a wrapper.
* **`grants`**: The `Action:Class:Domain` capabilities the attacker gains upon successful execution. These are what graph engines use to chain threats into multi-step attack paths - a threat's `grants` resolves another threat's `requires`.

**On Control entries:**

* **`denies`**: The `Action:Class:Domain` capabilities this control prevents the attacker from gaining. A control that denies a capability blocks every attack path downstream of it.
* **`overriddenBy`**: A logical condition expressing what attacker capability would bypass this control entirely. Used to model conditional control effectiveness and defence weaknesses.

### The CIANA Triad - `compromises` / `protects` / `cybersecurityProperties`

Threats, controls, and assets each carry an array drawn from the same 5-value CIAAN enum (`Confidentiality`, `Integrity`, `Availability`, `Authenticity`, `Non Repudiation`) - but under a different field name per entity type, because the *relationship* to the property differs:

* **Threat `compromises`**: which properties this threat's mechanism actually violates upon success (should agree with what its `grants` capability implies - e.g. a threat granting `Extract:Crypto` should list `Confidentiality`).
* **Control `protects`**: which properties this control's mechanism actually defends (should agree with its `denies`/`overriddenBy` - a purely detective control like an IDS protects different properties than a purely preventive one).
* **Asset `cybersecurityProperties`**: which properties this asset inherently *requires* to remain trustworthy, independent of any specific threat or control - the asset's own baseline security requirement.

The enum is shared because the concept (confidentiality, integrity, etc.) is the same; the field name is different because "what a threat breaks," "what a control defends," and "what an asset needs" are three distinct claims, not the same fact restated three times.

### Attack Potential Scoring

The `attackFeasibility.attackPotential` block on each threat entry (and the mirrored `attackFeasibilityReduction.attackPotentialReduction` block on each control entry) follows **ISO/SAE 21434 Annex I Table I.1**. Each of the five components must be assigned one of the following rating keys - the prefix names the dimension, the number names the tier within it: `ET0`-`ET4` (Elapsed Time, 5 tiers), `SE0`-`SE3` (Specialist Expertise, 4 tiers), `KoIC0`-`KoIC3` (Knowledge of the Item or Component, 4 tiers), `WoO0`-`WoO3` (Window of Opportunity, 4 tiers), `Eq0`-`Eq3` (Equipment, 4 tiers) - lower numbers mean easier-for-the-attacker on threats, and weaker-reduction on controls.

Catalog entries should be rated for the **complete attack scenario** they describe, using the highest plausible value for each component dimension. The platform that consumes these catalogs aggregates component scores, maps them to the ISO 21434 Annex I feasibility level (High / Medium / Low / Very Low), and cross-references with worst-case asset impact through ISO 21434 Table H.8 to produce a final risk rating. The on-device platform processes all data locally - no catalog or asset information is transmitted externally.

### Scalability

Every threat also carries a required `scalability` field - `Single Vehicle` | `Limited Batch` | `Fleet-Wide` - capturing the realistic breadth of exploitation the threat's *mechanism* supports, independent of any single company's actual fleet size. This is a different axis from `attackFeasibility`, which rates attacker effort *per instance*; `scalability` rates how many vehicles one successful exploit chain reaches without being independently repeated.

* **`Single Vehicle`**: requires unique, per-target physical possession or proximity that must be individually repeated for each additional victim (hardware fault injection, JTAG probing, chip decapsulation). Attacker effort scales linearly with victim count.
* **`Limited Batch`**: bounded by something narrower than the full fleet but broader than one vehicle - a specific production run, hardware revision, regional configuration, backend tenant/campaign, or an RF-proximity attack (jamming, relay, V2X spoofing) that only reaches whoever is physically in range at the moment of attack. Most MITRE-style follow-on techniques (persistence, discovery, defense evasion, lateral movement) fall here by default, since they operate within an already-compromised vehicle's boundary and inherit their real reach from whichever initial-access threat established that foothold, rather than being an initial-access vector themselves.
* **`Fleet-Wide`**: remotely or systemically replicable with no proximity or possession requirement - a backend/cloud API flaw, a vulnerability in shared firmware identical across every unit running it, an OTA/update-channel compromise, or a cryptographic weakness in a protocol used fleet-wide.

This field lives only on threats. A control's relevance to fleet-wide risk and an asset's realistic exposure both inherit scalability transitively via the capability graph - which threats can reach a given control or asset through matching `action:assetClass:domain` triples - rather than needing their own independent copy of this dimension.

### Threat Actors

Every threat also carries a required `threatActors` array (one or more values) capturing *who* would realistically want to execute the threat and *why* - a different axis again from `attackFeasibility` (how hard) and `scalability` (how far it reaches). Deliberately excludes skill-tier labels like "script kiddie": attacker sophistication is already captured by `attackFeasibility`'s `specialistExpertise`/`equipment` components, so this field is about motivation and access relationship, not capability.

* **`Nation-State / State-Sponsored`**: well-resourced, government-directed actors motivated by espionage, strategic disruption, or sabotage. Reserved for genuinely high-sophistication or high-value-target threats (root-of-trust material, supply-chain implants, backend infrastructure at scale) - not applied reflexively to every serious-sounding threat.
* **`Organized Cybercrime`**: financially motivated - vehicle theft rings, data-for-resale, fraud, extortion. The single most broadly-applicable category, since most threats have some plausible monetization path.
* **`Hacktivist`**: ideologically/politically motivated, favoring disruptive or visible outcomes over quiet financial gain.
* **`Insider (Employee / Dealer / Supplier)`**: someone abusing access obtained through an employment, dealership, or supply-chain *relationship* - not simply "an attacker who already has code execution from an earlier step in the chain," which applies equally to external attackers and isn't itself a relationship-based signal.
* **`Vehicle Owner / Authorized User`**: the vehicle's own legitimate possessor acting against the OEM's (or a regulator's/insurer's) interest - feature-unlock/DRM circumvention, odometer/emissions tampering, insurance/warranty fraud, disabling telemetry, or rolling back firmware to regain a removed feature.

These values converge from ENISA's Threat Landscape and industry threat-intelligence taxonomies (Nation-State, Organized Cybercrime, Hacktivist, Insider), extended with Vehicle Owner/Authorized User - the automotive-domain instantiation of NIST SP 800-30's "customer" adversarial threat-source category. Note that neither ISO/SAE 21434 nor SAE J3061 defines a threat-actor taxonomy of their own - 21434 explicitly routes around this by scoring attack feasibility instead, which this catalog already does; `threatActors` is a deliberate addition beyond the standard's own methodology, not a value drawn from it.

---

## Disclosure

### AI-Assisted Development

Portions of the threat, control, and asset catalog entries were developed with the assistance of Large Language Models (LLMs). All catalog content has been subject to expert human review against ISO/SAE 21434 requirements, automotive E/E architecture conventions, and established cybersecurity engineering practice.

Despite this review, given the scope and technical depth of the catalogs, **errors, inaccuracies, or omissions may be present**. Catalog entries should be validated against your specific vehicle architecture, threat landscape, and applicable regulatory requirements before use in a formal TARA. OpenASC is a reference foundation, not a substitute for qualified cybersecurity engineering judgement.

Corrections, improvements, and new catalog entries submitted via pull request are strongly encouraged and actively reviewed.

---

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## License

This project is licensed under the MIT License. See [LICENSE.md](LICENSE.md) for details.

## Author

[Stefanos Flokos](https://www.stefanosflokos.dev)