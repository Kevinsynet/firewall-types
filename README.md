# Different Types of Firewalls

- **Experiment ID:** EXP-FW
- **Experiment Name:** Different Types of Firewalls
- **Folder:** /experiments/firewall-types/
- **Entry File:** index.html
- **Navigation Title:** Types of Firewalls
- **Short Description:** Interactive study and simulation of different firewall architectures and traffic filtering.
- **Required Libraries:** None
- **Inputs:** Firewall type, packet parameters, firewall rules
- **Outputs:** Firewall decisions, explanations, comparison results, quiz scores
- **Expected Navigation Link:** /experiments/firewall-types/

## Setup and integration

Open `index.html` directly in a modern browser. No installation, build, backend or internet connection is required for the experiment. Keep `index.html`, `style.css`, `script.js` and this README in the same folder. To integrate, copy the folder to `/experiments/firewall-types/` and add a navigation link to that path. All assets use relative paths; no other modules are modified. Reference links alone require internet access.

## Features

Nine navigable sections: overview, expandable theory, procedure, simulator, interactive comparison, practical assessment, quiz, references and feedback. Responsive cream/terracotta design includes a companion panel, keyboard-accessible controls, reduced-motion support, progress tracking, inline SVG topology, animated packets, notifications and empty states.

The simulator supports six firewall architectures, ten packet/session inputs, six presets, editable ordered header rules, same-packet comparison, inspection traces and a timestamped, filterable log (latest 100 runs). Reset restores the packet, architecture and default rules; Clear log is separate. Click a scenario to populate and automatically run it. Changing the firewall reruns the current traffic. The topology scrolls into view for a 2.5-second, three-stage animation, with its verdict directly below. Replay simulation repeats the current packet. Manual packet fields and rule editing are collapsed under Advanced settings; Apply & run validates and executes changes. Scenario selection preserves custom rules. Rapid scenario changes cancel the previous animation/result, so stale results are not logged.

Each assessment draws five distinct scenarios from a 24-question bank, excluding all five from the previous set. Opening Assessment, reloading it, or selecting New questions starts a fresh set. Previous question IDs persist locally when storage is available. Practical predictions use default rules independently of simulator changes. Ten MCQs provide immediate explanations, a final score and answer review. Restart controls reset each exercise. Quiz answers and section visits persist using `exp-fw-` localStorage keys. Feedback is validated and saved only locally, with a clear control. Logs, edited rules and assessment answers are session-only. Section progress measures exploration, not mastery.

## Deterministic policy

Header rules match exact source/destination IPv4, protocol, destination port and direction. `*` means any address/port; `any` means any protocol/direction. Lowest priority number wins; equal priorities use row order. No match means deny. ICMP has no modeled ports: only wildcard-port rules match it.

Default policy: deny TCP/22 at priority 10; allow TCP/80 at 20; allow TCP/443 at 30; allow UDP/53 at 40. All directions and IPs match these default rules.

- **Packet filter:** header rules only; ignores payload and state.
- **Stateful:** rejects invalid state; established tracked flows bypass new-flow rules; new traffic uses header rules.
- **Circuit gateway:** header rules followed by valid TCP negotiation; rejects UDP and invalid/failed sessions in this model.
- **Proxy:** header rules followed by HTTP(S)-over-TCP support and a policy denying `/admin` and descendants. No generic malware detector.
- **NGFW:** stateful policy plus an HTTP/HTTPS/DNS application allowlist and rejection of any simulated threat flag. Threat checks also apply to established flows.
- **WAF:** independent web policy, without the network rule list. HTTP(S)-over-TCP is checked for SQL injection/XSS flags, decoded `OR 1=1`, `UNION SELECT` and script-tag patterns. Non-HTTP traffic passes through explicitly **outside inspection scope**; this does not mean it is safe. A generic malware flag alone is not a web attack signature in this model.

## Test cases and expected outputs

Use defaults unless a row says otherwise.

| Input | Architecture | Expected |
|---|---|---|
| Normal browsing, TCP/443 | All six | Allowed |
| New SSH TCP/22 | Packet / Stateful / Circuit / Proxy / NGFW | Blocked by header policy |
| SSH TCP/22 | WAF | Allowed, outside scope |
| DNS UDP/53 | Packet / Stateful / NGFW | Allowed |
| DNS UDP/53 | Circuit / Proxy | Blocked, unsupported session/application |
| DNS UDP/53 | WAF | Allowed, outside scope |
| HTTPS with malware flag | NGFW | Blocked by signature |
| HTTPS with malware flag | Packet | Allowed, headers only |
| Malicious HTTP preset | WAF / NGFW | Blocked |
| Established TCP/8443 | Stateful / NGFW | Allowed |
| Established TCP/8443 | Packet | Blocked, default deny |
| Invalid state, TCP/443 | Stateful / NGFW | Blocked |
| Failed TCP negotiation | Circuit | Blocked |
| HTTP(S) `/admin` | Proxy | Blocked |
| ICMP with default rules | Packet | Blocked |
| Priority 1 deny-all before defaults | Packet | Blocked |
| Invalid IPv4 or port outside 1–65535 | Any | Validation prevents run |

## Manual checklist

1. Open the local entry file offline; visit all nine sections and use keyboard navigation.
2. Click each preset and verify it runs automatically. Change architecture and verify automatic re-evaluation. Replay, reset, and switch scenarios during animation; confirm only the latest run finishes. Open Advanced settings and test Apply & run, including invalid inputs while the panel is collapsed. Compare all types and inspect reasons. Edit IP, port, action, direction and priorities; add/remove rules. Verify default-deny and equal-priority row ordering.
3. Filter Allowed/Blocked logs, clear them and reset/replay the workbench.
4. Complete five assessment predictions; verify each explanation against the displayed packet and default policy. Select New questions and verify five distinct questions with no overlap with the previous set; check that scoring resets. Reload and repeat.
5. Answer all ten quiz questions; inspect explanations and final review. Reload midway to check saved progress. Restart.
6. Submit short/blank feedback (rejected), save valid feedback, reload, then clear it. Test with browser storage disabled.
7. Inspect layouts at 375px, 768px and desktop widths; confirm all controls remain reachable and rule tables scroll.
8. Check developer console for errors and follow both reference links when online.

## Verification performed

JavaScript syntax validation passed. Sixteen automated engine assertions passed, covering header rules, priority, state, session validation, proxy policy, threat handling, encoded web attacks, WAF scope and ICMP. Browser visual/interaction QA was unavailable: the execution sandbox denied binding the preview server and the browser policy blocked `file:` URLs. The manual checklist above remains to be executed in a user browser.

## Limitations and sources

This is an educational model, not a production firewall or security scanner. It sends no network packets, implements no real connection tracking, and uses user-declared application identity and simulated threats. HTTPS inspection assumes decrypted content. IPv4 exact matching only; no CIDR, IPv6, source ports, real protocol parsing, identity, TLS handling, payload engine or persistent session table. Circuit UDP relay is intentionally omitted. Architecture features overlap in real products and policy choices vary. Browser-local storage availability and persistence for local files vary by browser. Feedback is not sent to an instructor or server.

Sources: [NIST SP 800-41 Rev. 1](https://csrc.nist.gov/pubs/sp/800/41/r1/final) for foundational firewall architecture/policy; [OWASP WAF introduction](https://community.owasp.org/Web_Application_Firewall) for HTTP inspection scope. The simulator policies are explicit teaching examples, not vendor implementations.

Guided simulator update: JavaScript syntax and automated control-flow checks passed for run/replay, canceled-run isolation, control unlocking, unique control IDs and collapsed Advanced settings. Browser visual QA remains unverified under the environment restrictions described above.

Packet journey update: packet positions are measured from the actual device centers and refreshed on resize. Sending, inspection and outcome stages show progress; blocked packets stop at the firewall, allowed packets disappear on server arrival. Reduced-motion preferences preserve stage feedback without motion. Automated geometry, allowed/blocked sequence and cancellation checks passed; visual browser verification remains pending.

## Guided learning additions

The existing nine sections, dashboard and brown/cream design remain. A four-step learning path connects Theory, Simulator, Assessment and Quiz. Simulator scenarios include everyday descriptions and optional allow/block predictions. Capability chips describe the selected educational model; unsupported capabilities are marked explicitly. Beginner mode keeps Advanced settings collapsed; Advanced mode opens them. A quick-help glossary explains packet, protocol, port, state, policy and threat terms.

Results explain what happened, why, and a suggested next experiment, with expandable inspection steps. Compare firewalls shows two independently selectable architecture panels using a fixed snapshot of the same packet and rules; all six results remain available in an expandable list.

Assessment and Quiz include a printable progress/completion report. It combines the latest completed assessment (saved locally) with the current completed quiz, lists missed topics and identifies incomplete work. Printing can also save a PDF through the browser print dialog. Starting a new assessment retains the previous completed assessment score until the new set finishes; restarting the quiz marks that component incomplete. The report is a learning record, not a certificate.

Verification: syntax checks and automated report checks passed for incomplete states, combined scores, missed topics, six scenario stories, capability mappings and differing packet-filter/NGFW verdicts. Browser visual QA remains pending. Manual checks: follow each guide link; predict and replay a scenario; switch modes and architectures; compare two types; complete assessment and quiz; verify report scores and print preview.
