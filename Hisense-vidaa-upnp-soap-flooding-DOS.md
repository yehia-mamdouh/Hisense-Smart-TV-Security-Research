# Hisense VIDAA Smart TV - UPnP AVTransport SOAP Flooding Denial of Service

> **Security research advisory | Availability impact | LAN-adjacent attack surface**

## Executive summary

A LAN-adjacent attacker can cause a **complete denial of service** on the tested Hisense VIDAA Smart TV by running a script that repeatedly sends unauthenticated `SetAVTransportURI` SOAP requests to the UPnP/DLNA AVTransport service on **TCP 38400**. During the researcher's flooding test, the TV **completely froze**, including the displayed content and normal device controls. **No notification, warning, confirmation dialog, or pairing popup appeared** to alert the user or request approval. The television did not recover through ordinary controls; restoring operation **required physically disconnecting power and reconnecting it to restart the TV**.

**Evidence qualification:** These hard-freeze and no-popup observations are reported by the researcher for the flooding test. The separate supplied console screenshot documents an outbound callback experiment with no HTTP callbacks; it is not direct evidence of the hard freeze.

## Tested environment

| Field | Value |
|---|---|
| Vendor / platform | Hisense / VIDAA |
| Device model | 55A57EXVT |
| Tested firmware | V0000.01.00a.P0220 |
| Affected component | UPnP/DLNA AVTransport SOAP service |
| Network port | TCP 38400 |
| Relevant SOAP action | `urn:schemas-upnp-org:service:AVTransport:1#SetAVTransportURI` |
| Attack position | A host able to reach the television's LAN-facing service |

## 01 - Vulnerability description

The Hisense VIDAA Smart TV exposes a UPnP/DLNA AVTransport SOAP service on **TCP port 38400** that accepts `SetAVTransportURI` requests from a network-reachable client without authentication. When i have executed a python script issuing these requests repeatedly, **the entire television became unresponsive**, rather than merely stopping a media stream or crashing one application.

The display froze on its current content, and **both the remote control and the television's physical power control stopped responding**. The TV presented **no warning, notification, consent request, or pairing popup** before or during the attack. The device remained frozen until power was physically disconnected and reconnected, forcing a hard restart.

The observed failure is a **device-wide availability loss triggered remotely over the local network**. Insufficient request throttling or isolation of AVTransport processing is a plausible cause, but the precise internal failure mechanism has not been verified through firmware traces.

## 02 - Attack-path diagram

```mermaid
flowchart TD
    A[LAN-adjacent network client] --> B[TCP 38400: UPnP AVTransport SOAP endpoint]
    B --> C[Repeated SetAVTransportURI requests]
    C --> D[AVTransport command processing]
    D --> E{Resource protection / rate limiting sufficient?}
    E -->|Reported vulnerable behavior| F[Entire TV freezes without warning or popup]
    F --> G[Remote and physical power controls stop responding]
    G --> H[Disconnect and reconnect mains power to recover]
    E -->|Protected behavior| I[Requests rejected, limited or handled without UI loss]
```

*The diagram illustrates the reported attack path, not a verified internal implementation trace.*

## 03 - Security impact

### 3.1 - Complete remote denial of service

**Impact: Total loss of TV availability.** Once I executed the flooding script against the tested device, the television **froze completely** and became entirely unresponsive. The screen remained stuck on its current content, and neither the remote control nor the TV's physical power button responded. The user could no longer interact with the television, switch applications or inputs, or perform any normal operations.

**What an attacker can do:** A network-adjacent attacker who can reach TCP port 38400 can intentionally disrupt the television's operation by sending repeated unauthenticated SOAP requests, without needing credentials or physical access.

### 3.2 - No user notification or consent prompt

The TV displayed **no notification, warning, pairing request, or consent popup** when the attack began. The user therefore received no on-screen opportunity to approve, reject, or interrupt the incoming commands before the device froze.

**Security consequence:** The attack can begin silently from the local network and cause a sudden loss of availability without an on-screen indication of the cause.

### 3.3 - Forced physical power-cycle recovery

During the freeze, **the remote control and the TV's physical power button were unresponsive**. The television could not be recovered using normal controls. **I had to unplug the TV from mains power and reconnect it to restart the device.**

**Security consequence:** The attack causes a disruptive device-wide failure requiring physical intervention, rather than a temporary playback interruption or an application restart.

### 3.4 - Scope and limitations

The observed effect concerns **availability**. The supplied materials do not demonstrate data theft, operating-system command execution, or persistence. Testing on other Hisense models or firmware versions has not been established. The separate callback screenshot does not itself demonstrate the hard-freeze behavior.

## 04 - Research evidence

### 4.1 - Original flooding-test observations (researcher advisory)

The supplied advisory records these observations:

- Initial SOAP requests reportedly received HTTP `200` responses.
- The flooding requests completely froze the TV, requiring a physical power disconnect and restart to **restore functionality.**.
- Remote-control and physical-button responsiveness were reportedly lost.
- The tested freeze required disconnecting and reconnecting mains power to restore operation.
- No warning, notification, consent prompt, or pairing popup was shown to the user.

## 05 - Reproduction methodology (high level)

In an isolated, authorized laboratory environment, establish baseline TV responsiveness and AVTransport SOAP response times, then compare them against a controlled, bounded series of `SetAVTransportURI` requests. Record whether the on-screen UI, remote control, and physical controls remain responsive. Preserve request/response timestamps, device video, packet captures, and recovery steps. Stop immediately if responsiveness degrades.

**Public Disclosure Note**: I confirmed that sustained UPnP SOAP flooding caused the television to freeze completely, requiring a physical power restart. To prevent misuse, I have omitted unrestricted flooding commands and live target addresses from this public advisory.

## 06 - Root cause assessment

**Hypothesis:** Inadequate request-rate controls and/or failure to isolate AVTransport processing from UI-critical resources may permit repeated commands to exhaust or block shared execution capacity.

**Confidence:** The original advisory attributes the issue to missing throttling and an overloaded request queue. The provided materials do not contain instrumentation proving that particular implementation mechanism. Vendor investigation is required to confirm the root cause.

## 07 - Recommended remediation

1. Require appropriate authorization or pairing for state-changing AVTransport commands.
2. Apply per-client and global request-rate limits, queue bounds, and backpressure.
3. Ensure malformed or excessive SOAP requests cannot block UI-critical threads.
4. Add command timeouts and safe cancellation for stalled media operations.
5. Restrict UPnP control interfaces to trusted network segments where possible.
6. Regression-test recovery of remote-control and power-management responsiveness during network request bursts.

**Temporary mitigation:** Isolate the TV from untrusted LAN clients and filter access to TCP 38400 where network controls permit, while preserving required legitimate features.

## 08 - Disclosure and validation status

| Item | Status |
|---|---|
| Original researcher advisory | Provided |
| Hard-freeze behavior | Researcher reports complete freeze requiring power disconnection; independent visual/packet evidence not attached here |
| Callback test | Completed; no HTTP callbacks observed |
| Vendor acknowledgement / patch | Not established in supplied materials |
| CVE assignment | Not supplied |

**Research scope:** Hisense 55A57EXVT, VIDAA firmware V0000.01.00a.P0220. Findings should not be generalized to all Hisense or VIDAA products without additional testing.

---

*Responsible disclosure: This document is intended to support coordinated vulnerability reporting and reproducible defensive validation.*
