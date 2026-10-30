# Unauthorized Content Injection via DLNA AVTransport on Hisense VIDAA Smart TV

**Security Research Advisory** · **Yehia Elghaly** · **Coordinated Disclosure**

> [!IMPORTANT]
> **CVE ID pending.** This document describes a vulnerability verified on one Hisense 55A57EXVT running firmware V0000.01.00a.P0220.

| Field | Tested configuration |
|---|---|
| Vendor / platform | Hisense / VIDAA OS |
| Device | Hisense 55A57EXVT |
| Firmware | `V0000.01.00a.P0220` |
| Affected service | UPnP/DLNA AVTransport SOAP, TCP `38400` |
| Attack prerequisite | Network access to the AVTransport service; one-time acceptance of a DLNA consent popup for on-screen playback in the demonstrated workflow |

## Executive Summary

The tested Hisense VIDAA Smart TV accepts media URLs supplied by a network client through the DLNA AVTransport `SetAVTransportURI` interface. Following the one-time on-screen DLNA authorization in the tested workflow, an attacker on the local network can cause the TV to fetch and display attacker-hosted media **without further confirmation for each command**.

I have demonstrated HTTP requests from the TV to a controlled content server and photographed attacker-controlled content rendered on the physical television. The primary security impact is **unauthorized control of displayed media**, which can interrupt normal viewing and support deceptive or misleading on-screen content.

## Root Cause & Attack Surface

The AVTransport SOAP service processes client-supplied media URIs without sufficiently restricting the origin of media or enforcing authorization for every subsequent playback action. The research notes that URL fetching can begin before the user responds to the initial consent popup; however, **successful visible playback without that initial consent was not established**.

## Demonstrated Security Impacts

### 01 - Remote Injection of Attacker-Controlled Content onto the TV Screen

A network-adjacent client can direct the television to retrieve a media resource from an attacker-controlled HTTP server and, following the initial DLNA approval, render supported content on the physical screen. The accompanying physical-device evidence shows a generated content-injection display and a media playback screen.

**Attacker impact:** An unauthorized network client could replace or interrupt what the viewer is watching with arbitrary *supported media*, including deceptive notices or impersonated security messages embedded as images or video. This enables on-screen social engineering and interference with normal television use.

<img width="1419" height="1109" alt="71a1d1b8-11cb-4744-9056-f772c38c8e81" src="https://github.com/user-attachments/assets/5dbd56f1-2d14-465b-99f6-0171488af48f" />

*Figure 1 - Physical television displaying researcher-generated content-injection proof of concept.*

<img width="1472" height="1069" alt="abc14a47-549f-4bce-878d-8f9f1ce6b54c" src="https://github.com/user-attachments/assets/5937fa3f-89c9-4edd-a685-86df2f156a6c" />

*Figure 2 - Physical television displaying test video content via the media playback interface.*

### 02 - Untrusted Media Fetching and Playback Control

The test console records successful `SetAVTransportURI`, `Play`, and `Stop` operations (HTTP 200) and confirms that the television fetched resources from the researcher's HTTP server. The log records requests for an H.264 MP4, a JPEG image, and an HTML document.

**Attacker impact:** A network client able to invoke the AVTransport commands can make the television request externally hosted content and initiate or stop playback. This could disrupt active viewing or expose the TV to attacker-selected media resources.

<img width="1280" height="851" alt="Screenshot from 2026-10-09 19-19-41" src="https://github.com/user-attachments/assets/39f9cbbc-4a0b-41df-bc61-df914549b878" />

*Figure 3 - SOAP command results and server-side fetch logs for video, image, and HTML test resources.*


### 03 - Persistent Control After Initial DLNA Approval

The supplied advisory reports that once the device's DLNA approval prompt is accepted, later media commands can execute without an additional user prompt for each resource.

**Attacker impact:** A previously approved local-network client may repeatedly change or interrupt the displayed media without renewed confirmation, creating an ongoing integrity and availability risk until that client's access is revoked or otherwise blocked.

## Attack-Path Overview

```mermaid
flowchart TD
    A[Network-adjacent attacker] --> B[Reachable DLNA AVTransport service]
    B --> C[Submit attacker-hosted media URI]
    C --> D[TV processes URI and fetches resource]
    D --> E{Initial DLNA consent for playback?}
    E -->|Accepted| F[AVTransport Play command]
    E -->|Not accepted| G[Visible playback not demonstrated]
    F --> H[Supported media displayed on TV]
    H --> I[Viewing interruption or deceptive on-screen content]
    F --> J[Subsequent commands without per-item approval]
    J --> H
```

**Scope:** The diagram distinguishes the observed media fetch from the separate user-consent requirement for demonstrated screen playback.

## Remediation Recommendations

- Require explicit, revocable authorization for remote media-control clients, with appropriate user visibility.
- Validate and constrain AVTransport URI schemes and destinations; restrict access to local files and internal services.
- Do not fetch or process untrusted media URLs before the required authorization decision.
- Provide users with controls to review and revoke paired DLNA devices; consider confirmation for disruptive playback changes.
- Verify fixes across affected firmware versions and TV models.
