[README-Hisense-Smart-TV-Security-Research.md](https://github.com/user-attachments/files/33917572/README-Hisense-Smart-TV-Security-Research.md)
<div align="center">

# 📺 Hisense Smart TV Security Research

### Six security findings across VIDAA's network-facing control and media services

**Independent vulnerability research and vendor notification**

![Research](https://img.shields.io/badge/Research-Smart%20TV%20Security-111827?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Hisense%20VIDAA-0f766e?style=for-the-badge)
![Findings](https://img.shields.io/badge/Findings-6-b91c1c?style=for-the-badge)
![Status](https://img.shields.io/badge/Disclosure-No%20Vendor%20Response-f59e0b?style=for-the-badge)

**One television. Multiple exposed services. A shared security boundary: the local network.**

[Research Overview](#-research-at-a-glance) · [Six Findings](#-the-six-findings) · [Attack Surface](#-attack-surface-overview) · [Disclosure Timeline](#-vendor-notification-timeline)

</div>

---

## 🔎 Research at a Glance

This repository documents **six security findings** identified during independent security testing of a **Hisense VIDAA Smart TV**. The research examines how network-accessible automation, discovery, media playback, remote-control, and messaging interfaces may allow a network-adjacent actor to interact with the television without adequate authentication or authorization.

The observed consequences span **browser-context JavaScript execution, application-data exposure, on-screen content manipulation, remote app control, denial of service, and TV control-plane weaknesses**.

| Research detail | Tested environment |
|:--|:--|
| **Manufacturer / platform** | Hisense / VIDAA |
| **Tested model** | `55A57EXVT` |
| **Tested firmware** | `V0000.01.00a.P0220` |
| **Browser component** | Google Cobalt `19.lts.5.205289-qa` |
| **Attacker position** | Network-adjacent, with reachability to the relevant exposed service |
| **Findings documented** | **6** |

> **Scope:** Results are based on the specific device and firmware tested. They should not be assumed to affect every Hisense television or VIDAA release without additional verification.

## 🧭 The Six Findings

| # | Vulnerability / exposed service | Security impact | Evidence summary |
|:--:|:--|:--|:--|
| **01** | [01 - Cobalt WebDriver Exposure](Unauthenticated%20WebDriver%20Access%20via%20QA%20Build%20in%20Production%20Firmware%20in%20Google%20Cobalt.md)· TCP `4444` | Browser-context JavaScript execution, application storage disclosure, external HTTP data exfiltration, UI manipulation | Existing WebDriver session accessed; internal storage extracted; outbound webhook POST observed |
| **02** | [02 - DIAL Application Control](Unauthenticated%20App%20State%20Surveillance%20and%20Remote%20App%20Control%20via%20DIAL.md)· TCP `56789` | Unauthorized app-state inspection, application launch, and termination | DIAL state queried; app launch returned HTTP `201`; app termination returned HTTP `200` |
| **03** | [03 - DLNA Content Injection](Unauthorized%20Content%20Injection%20via%20DLNA%20AVTransport%20on%20Hisense%20VIDAA%20Smart%20TV.md)· TCP `38400` | Unwanted on-screen media and content presentation; playback interference | Media fetches logged and test content displayed on physical TV; consent behavior varies by workflow |
| **04** | [04 - Single-Request Denial of Service](Unauthenticated%20Single-Request%20Denial%20of%20Service%20via%20UPnP%20AVTransport%20on%20Hisense%20VIDAA.md) · TCP `38400· | Complete TV unresponsiveness requiring a physical power-cycle, as reported in the freeze test | I confirmed a complete TV freeze with no response to the remote or physical power button; restoring operation required disconnecting and reconnecting mains power |
| **05** | [05 - UPnP Authorization Weakness](Hisense%20VIDAA%20Smart%20TV%20-%20Unauthorized%20UPnP%20Control%20and%20Consent-Popup%20Enforcement%20Weakness.md)· TCP `38400` | Unauthorized override of remote-control functions, including volume modification; playback-state exposure and command processing during a consent popup | Volume changes verified by readback; SOAP commands returned success while popup was visible |
| **06** | [06 - MQTT Authentication Weaknesses](Hisense%20VIDAA%20Smart%20TV%20-%20MQTT%20Remote-Control%20Authentication%20Bypass%20and%20PIN%20Brute-Force%20Vulnerabilities.md)· TCP `36669` / TLS | Broker access using shared credentials; volume-control command submission; rapid PIN guessing and apparent pairing success | MQTT connection established; `7,005` PIN attempts in `16.7 s`; apparent success response observed |

**Read the individual advisory files in this repository for the full technical descriptions, reproduction evidence, screenshots, and scope limitations.** The file list appears above this README on the repository's **Code** tab.

### What makes these findings significant?

- **Confidentiality:** The WebDriver finding includes demonstrated extraction of JavaScript-accessible TV data and receipt of that data at an external HTTP endpoint.
- **Integrity:** Network-accessible control interfaces can alter the viewer's experience through application control, browser UI changes, media presentation, or volume adjustment.
- **Availability:** My SOAP flooding test rendered the television completely unresponsive until I physically disconnected and reconnected its power supply.
- **Authentication boundaries:** Several interfaces rely on absent, shared, incomplete, or inconsistently enforced authorization mechanisms. Their requirements differ; not every finding is a zero-interaction exploit.

## 🕸️ Attack Surface Overview

```mermaid
flowchart TB
    A["Network-adjacent actor"] --> B["Hisense VIDAA Smart TV"]

    B --> W["Cobalt WebDriver<br/>TCP 4444"]
    B --> D["DIAL<br/>TCP 56789"]
    B --> U["UPnP / DLNA / SOAP<br/>TCP 38400"]
    B --> M["MQTT Control Broker<br/>TCP 36669 / TLS"]

    W --> W1["JavaScript in browser context"]
    W1 --> W2["Application storage access<br/>and external exfiltration"]
    W1 --> W3["UI manipulation"]

    D --> D1["App-state inspection"]
    D --> D2["Launch / stop apps"]

    U --> U1["Unwanted media presentation"]
    U --> U2["Volume / playback control"]
    U --> U3["Confirmed hard-freeze DoS"]

    M --> M1["Shared-credential broker access"]
    M --> M2["Rapid PIN guessing"]
    M2 --> M3["Potential expanded control<br/>requires action-level verification"]

    classDef actor fill:#7f1d1d,color:#ffffff,stroke:#ef4444,stroke-width:2px;
    classDef device fill:#1e3a8a,color:#ffffff,stroke:#60a5fa,stroke-width:2px;
    classDef surface fill:#164e63,color:#ffffff,stroke:#22d3ee;
    classDef impact fill:#334155,color:#ffffff,stroke:#94a3b8;
    classDef caution fill:#78350f,color:#ffffff,stroke:#fbbf24;
    class A actor;
    class B device;
    class W,D,U,M surface;
    class W1,W2,W3,D1,D2,U1,U2,U3,M1,M2 impact;
    class M3 caution;
```

> **Diagram note:** This is a summary of distinct findings, not a single chained exploit. The prerequisites and degree of demonstrated control differ across services.

## 🧪 Evidence and Validation

Each individual finding is documented separately to preserve its specific attack prerequisites, observed behavior, and evidence. Supporting material includes, where applicable:

- Network service responses and protocol-level test output.
- On-device photographs of the television displaying researcher-controlled content.
- Browser storage extraction and external webhook receipt.
- Command execution and device-state verification logs.
- Screenshots of authentication and consent behavior.

## 📬 Vendor Notification Timeline

| Date | Event |
|:--|:--|
| **27 April 2026** | Initial vulnerability report sent to **Hisense**. |
| **2 May 2026** | Follow-up reminder sent to the vendor. |
| **30 October 2026** | **No vendor response received more than six months after my initial report.** |

**Vendor status:** As of **30 October 2026**, I have received no acknowledgement or response from Hisense since my initial notification on 27 April 2026.

This timeline documents my communications with Hisense. I have not received confirmation of the findings, a severity assessment, or remediation information from the vendor. I will update the advisories if a response becomes available.

## 🛡️ Security Recommendations

At a high level, vendors and integrators should consider:

1. **Disable production exposure of debugging and automation services** or restrict them to authenticated, explicitly enabled development environments.
2. **Require device-specific authentication and authorization** for remote-control, media, and application-management commands.
3. **Enforce meaningful user consent** before sensitive actions and ensure consent prompts actually block command execution until approval.
4. **Rate-limit pairing attempts**, bind pairing approval to a requesting client, and enforce lockouts or challenge expiration.
5. **Harden UPnP/DLNA processing** against untrusted requests and resource-exhaustion conditions.
6. **Reduce unnecessary network exposure**, use broker topic-level access controls, and protect communication with correctly validated TLS identities.

For owners and network administrators, **isolating Smart TVs from untrusted LAN clients** can reduce exposure while vendor fixes are unavailable.

## ⚖️ Responsible Research and Usage

This repository is published for defensive security research, vulnerability analysis, and responsible public disclosure following vendor notification. All testing was performed against a device under my control. Do not test systems without authorization.

**No universal exploitability claim is made:** The documented results apply to the tested configuration unless additional affected versions are established.

---

<div align="center">

### 📺 Smart TVs are computers on your network — their control interfaces deserve the same security scrutiny.

**Hisense Smart TV Security Research** · Independent Security Research · Vendor Notified

</div>
