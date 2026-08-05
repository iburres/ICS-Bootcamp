# Wireless Systems in ICS — Discussion Guide

**Format:** 20 open-ended questions for guided classroom dialogue. Each question is
phrased to invite several different answers rather than one "correct" response —
use them to get multiple students talking before you weigh in. Every question is
followed by **Instructor Reference** notes: talking points to prime the discussion,
redirect it if it stalls, or supply an angle nobody raised. Treat these as a crib
sheet, not a script — the goal is dialogue, not recitation.

Suggested use: pose the question, take 2–3 student answers before adding anything
yourself, then use the reference notes to introduce whichever angle the class
missed.

---

## 1. Why does adding wireless to an ICS/OT network change the risk model compared to a wired-only plant?

**Instructor Reference:**
- Wireless collapses the "you'd need physical access to the cable" assumption a lot of OT security has quietly relied on — an attacker in the parking lot or on the public road outside the fence line now has a path in.
- The broadcast nature of RF means traffic is available to anyone with an antenna in range, not just someone tapping a specific wire.
- Many OT wireless protocols (WirelessHART, Zigbee, Z-Wave) were adapted from consumer/building-automation designs, not purpose-built for adversarial industrial environments the way, say, DNP3 Secure Authentication was.
- It also changes the *safety* calculus — a wireless E-stop or safety interlock now depends on RF availability, which is a new failure mode a purely wired safety system never had.

---

## 2. What electromagnetic/physical-layer properties of 2.4 GHz Wi-Fi make it a good — or a risky — fit for an industrial plant floor?

**Instructor Reference:**
- 2.4 GHz penetrates non-metallic obstacles reasonably well but is heavily attenuated by steel enclosures, rebar-reinforced concrete, and large metal process equipment — common in plants — leading to dead zones and, ironically, to operators boosting power or adding repeaters that widen the attacker's usable range.
- It shares spectrum with Bluetooth, Zigbee, and industrial ISM-band equipment (some VFDs and induction heating systems radiate broadband noise right in this band), so the noise floor is already high before an attacker does anything.
- 5 GHz has more clean channels and less interference but shorter range and worse penetration — a real coverage-vs-security tradeoff, not just a bigger-number-is-better upgrade.
- Multipath reflection off metal tanks/piping can make signal strength an unreliable indicator of an attacker's actual distance — "the signal looks weak" doesn't mean "the attacker is far away."

---

## 3. How could an attacker learn something useful about an ICS wireless network without ever associating to it or sending a single packet?

**Instructor Reference:**
- Passive RF reconnaissance: SSID broadcasts, beacon frame vendor OUIs, and 802.11 probe requests can fingerprint the equipment vendor (and sometimes the exact model) before any handshake happens.
- Traffic *timing and volume* analysis — even encrypted traffic reveals polling intervals, burst patterns tied to shift changes, or process-cycle periodicity, all useful for reconnaissance.
- This is the entry point for framing "side-channel" thinking for the rest of the session: information leaks through metadata even when payload content is protected.

---

## 4. What is a side-channel attack, in general terms, and how might RF emissions specifically leak information about a physical process without anyone breaking encryption?

**Instructor Reference:**
- A side channel is any unintended information leak from *how* a system operates — timing, power draw, EM emissions, sound, heat — rather than from breaking its intended logic or cryptography.
- Correlate wireless transmission *timing/power* with the physical process: a wireless vibration sensor that transmits more frequently when vibration crosses a threshold tells an eavesdropper the machine's operating state, even if the payload itself is encrypted.
- Bring up TEMPEST/compromising emanations here as a segue if it doesn't come up on its own — unintentional EM radiation from the ICS equipment itself (PLCs, VFD drive cables) is a side channel that exists whether or not the plant uses wireless at all.

---

## 5. If you had a Software Defined Radio (SDR) and wanted to intercept or replay traffic from one of the ICS wireless protocols we're covering, which would be the easiest target and why?

**Instructor Reference:**
- Z-Wave (sub-1 GHz, historically weaker encryption in older S0 security class devices) and early Zigbee deployments (network key sometimes transmitted in the clear during device commissioning/pairing) are commonly cited as easier targets than modern WirelessHART.
- Cheap SDR hardware (RTL-SDR ~$25, HackRF ~$300) covers most of these bands; sub-1 GHz sometimes needs an upconverter, but that's a minor cost barrier, not a real one.
- WirelessHART's TDMA time-slotting + channel hopping makes casual interception harder — you need to track the hopping pattern, not just tune to one frequency — a good point of comparison for question 8.
- Push students to distinguish "interception" (need timing/decoding know-how) from "replay" (may not even need to decode the payload if you can capture and retransmit the raw RF burst).

---

## 6. Bluetooth shows up in ICS-adjacent contexts more than people expect — configuring VFDs, wireless HART adapters, HVAC controllers, handheld calibration tools. What's the actual risk, given Bluetooth has "real" encryption?

**Instructor Reference:**
- BlueBorne (2017) showed Bluetooth stack vulnerabilities that required *no pairing at all* — the mere presence of Bluetooth radio was the attack surface, which is a good example of encryption not mattering if the flaw is beneath it in the stack.
- Legacy PIN pairing (fixed 4-digit PINs, often "0000" or unchanged defaults on commissioning tools) is brute-forceable; Secure Simple Pairing (SSP) closes some of this but adoption in industrial handheld devices lags consumer devices by years.
- The real-world risk is often less "someone hacks the Bluetooth crypto" and more "the technician's Bluetooth-paired configuration tablet is also connected to the plant Wi-Fi and email" — Bluetooth as a lateral-movement bridge, not the primary target.

---

## 7. Zigbee uses AES-128, which sounds strong. Where does Zigbee security actually tend to fail in practice?

**Instructor Reference:**
- Key management, not the cipher: many implementations transmit the network key unencrypted during device join/commissioning — anyone sniffing that one join event gets the key for the whole mesh.
- Widespread key reuse — some vendors ship devices with the same default Trust Center link key across an entire product line.
- Zigbee's mesh/repeater design means compromising one weak device (a smart bulb, say, in a building-automation-adjacent ICS deployment) can be a foothold into the whole mesh, including more sensitive nodes.
- Good moment to connect to the building-automation/BACnet material from earlier in the course — Zigbee HVAC sensors are exactly the kind of device that blurs the IT/OT boundary.

---

## 8. WirelessHART uses TDMA time-slotting and channel hopping specifically for *reliability* in noisy industrial RF environments. Does that design choice also buy any security benefit, or is that a common misconception?

**Instructor Reference:**
- It raises the bar for casual/opportunistic eavesdropping (you need to follow the hop sequence, which requires knowing the shared hopping pattern/session info), but it is fundamentally an interference-avoidance mechanism, not a cryptographic one.
- WirelessHART does layer real security on top (join-key based network admission, AES-128-CCM* encryption, message integrity codes) — the hopping is a nice side benefit for a determined attacker's cost, not the actual security control.
- Good spot to explicitly flag the "spread spectrum = secure" misconception for question 16 — students often conflate the two.

---

## 9. Z-Wave operates sub-1 GHz (908.42 MHz in the US) instead of 2.4 GHz. What's the security-relevant tradeoff of that frequency choice?

**Instructor Reference:**
- Sub-1 GHz penetrates walls/obstacles better and has a longer effective range per unit of transmit power than 2.4 GHz — good for building-scale coverage, but that also extends the attacker's usable eavesdropping/injection range outside the building.
- Less spectrum crowding (fewer coexisting consumer devices) means less ambient noise to hide in, but also means anomalous transmissions are easier to spot with a spectrum analyzer if you're defending, not just attacking.
- Historically, Z-Wave's S0 security class had a well-documented key-exchange downgrade vulnerability; S2 (introduced 2017) fixed the key exchange with ECDH — worth asking students whether they'd assume "Z-Wave is secure" as a blanket statement, or whether the security *class* of the specific device matters.

---

## 10. How would you actually jam a wireless ICS network at the physical layer, and is jamming always just a denial-of-service move, or can it enable something worse?

**Instructor Reference:**
- Physical principle: flood the receiver's noise floor near/above the legitimate signal's power at the receiver, dropping the signal-to-noise ratio below the threshold the demodulator needs — the attacker doesn't need to understand the protocol at all, just overpower it in-band.
- Straightforward DoS use: force a wireless-dependent safety interlock or E-stop into a fail state — ask the class whether that fails safe or fails dangerous, and why that depends entirely on the system design.
- The "worse" angle: selective/reactive jamming can force a device to fall back to a less-secure mode (e.g., an unencrypted commissioning/pairing state) or to a different channel/frequency an attacker is already positioned to intercept — jamming as a precursor to a downgrade or man-in-the-middle attack, not just an end in itself.

---

## 11. If an attacker captures a single wireless command — say, a wireless valve-actuator "open" command or a remote E-stop reset — what stops them from just replaying it later?

**Instructor Reference:**
- Rolling codes / one-time-use sequence numbers (like garage door openers evolved to use after early fixed-code models were trivially replayable) — each valid command can only be used once.
- Timestamps with a tight validity window plus synchronized clocks — a replayed command outside the window is rejected, though this pushes the problem toward "how do you protect the time sync itself" (nice callback to question 13).
- Cryptographic nonces/challenge-response so the receiver expects a value that changes every exchange, not just an encrypted-but-static payload.
- Ask directly: does simply *encrypting* the command prevent replay? (No — an attacker doesn't need to decrypt a captured ciphertext to resend it verbatim if there's no freshness check.)

---

## 12. Could you tell a legitimate ICS wireless transmitter apart from a cloned/spoofed one, even if the spoofed device sends bit-for-bit identical protocol messages?

**Instructor Reference:**
- RF fingerprinting / physical-layer authentication: real radio hardware has unique, hard-to-replicate analog imperfections — power-on transient shape, I/Q imbalance, carrier phase noise, clock drift — that can act like a hardware fingerprint independent of the protocol data.
- This is an active research area for ICS/IoT device authentication precisely because higher-layer credentials (keys, certificates) can be extracted or cloned, but reproducing another device's exact analog RF signature is much harder.
- Good discussion prompt: is this practical for a plant to deploy today, or mostly a research/high-value-asset technique? (Mostly the latter — cost and complexity are real barriers.)

---

## 13. PMUs (Phasor Measurement Units) on the power grid rely on GPS for precise time synchronization. What happens if that GPS signal is spoofed, and why does that count as a "wireless ICS attack" even though it has nothing to do with Wi-Fi/Bluetooth/Zigbee?

**Instructor Reference:**
- GPS/GNSS spoofing feeds a receiver a fabricated but plausible-looking satellite signal, shifting its computed time (and/or position) without the receiver detecting anything is wrong.
- PMUs depend on sub-microsecond-accurate timestamps to compute phase angle differences across the grid — corrupt the timing and you corrupt situational awareness data that grid operators use for stability decisions, without touching a single protocol packet.
- Reinforces the session's broader point: "wireless" as an ICS attack surface is bigger than the named short-range protocols — GPS, cellular, and even satellite links all count.

---

## 14. A plant deploys Wi-Fi for wireless HMI tablets and engineering laptops on the plant floor. What's the rogue-AP / evil-twin risk, specifically in this context?

**Instructor Reference:**
- An attacker broadcasts an AP with the same SSID (and often stronger signal, since they can place it closer or use more power) — devices configured to auto-join a known SSID connect to the attacker instead of the legitimate AP.
- Once associated, the attacker can intercept credentials, inject malicious content into an HMI web session, or pivot toward the actual OT network if the wireless segment isn't properly firewalled from it.
- Ask: does WPA2/WPA3 encryption prevent this? (No — the evil twin can run its own legitimate-looking encrypted network; the vulnerability is in *device trust of the SSID*, not the cipher.)

---

## 15. Even with all wireless radios disabled, PLCs, VFDs, and other ICS equipment still emit unintentional electromagnetic radiation. How could that be exploited, and what's this phenomenon called?

**Instructor Reference:**
- TEMPEST / compromising emanations — unintentional RF or conducted emissions from digital electronics (switching power supplies, backplanes, display cables) can be captured and, with enough signal processing, reconstructed into meaningful information (a technique historically called Van Eck phreaking for video signals).
- For ICS specifically: emissions correlated with I/O switching, backplane bus activity, or drive-cable PWM patterns can leak information about a control process's state to a sufficiently equipped and positioned adversary — no wireless radio involved on the victim's side at all.
- Worth noting this is a nation-state-level threat in most realistic risk models — expensive equipment and proximity required — but it completes the "wireless is bigger than the radios you turned on" theme from question 13.

---

## 16. Students sometimes hear "frequency hopping spread spectrum" and assume that means "encrypted" or "secure." Why is that a misconception?

**Instructor Reference:**
- Spread spectrum (frequency-hopping or direct-sequence) was originally developed for interference resistance and, in some military contexts, anti-jam resilience — not confidentiality.
- If the hopping sequence/pattern is derivable or is itself transmitted/negotiated in a predictable way, a receiver that knows the pattern can follow it just as easily as the intended receiver — hopping alone doesn't provide cryptographic secrecy.
- Real confidentiality still has to come from an actual cipher (AES, etc.) layered on top — this ties directly back to question 8's WirelessHART discussion, which is worth explicitly cross-referencing here.

---

## 17. What physical and administrative controls can reduce wireless ICS risk, and what makes them hard to actually deploy on a real plant floor?

**Instructor Reference:**
- RF shielding / Faraday-style enclosures for the most sensitive equipment — but this conflicts with the very reason wireless was chosen (accessibility, retrofit convenience, avoiding costly cable runs), so it's rarely applied everywhere.
- Continuous spectrum monitoring for rogue transmitters or anomalous RF activity — effective, but requires dedicated tooling and trained staff to interpret, which many plants don't budget for.
- Site RF surveys before deployment to understand actual coverage/leakage boundaries — commonly skipped because it costs time and money up front and wireless "just works" well enough to pass a quick functional test.
- Segmentation: keeping wireless-connected devices on their own firewalled VLAN/zone rather than flat access to the OT network — a policy control, not a physical one, and often the most cost-effective single mitigation.

---

## 18. Remote RTUs and pipeline telemetry sites often use cellular (LTE/5G) instead of local short-range wireless. What unique attack surface does that introduce?

**Instructor Reference:**
- SIM cloning/swapping can let an attacker impersonate the RTU's cellular identity to the carrier network.
- IMSI catchers ("Stingrays") can intercept or downgrade a cellular connection to a weaker/older standard (e.g., forcing a fallback from LTE to 2G, which has much weaker or broken cryptography) in the vicinity of a remote site.
- SS7 signaling vulnerabilities in the carrier's own core network can, in some documented cases, allow location tracking or call/SMS interception entirely outside the target's control — an attack surface the plant operator doesn't own or control at all, which is a distinct risk category from the local RF protocols discussed earlier.
- Ask: who is responsible for securing this layer — the plant, or the cellular carrier? (Usually neither fully addresses it — a real gap.)

---

## 19. How could a drone change the practical risk calculus for wireless ICS attacks that would normally require close physical proximity (Bluetooth range, Zigbee mesh range, etc.)?

**Instructor Reference:**
- A drone extends an attacker's effective antenna position to places a person on foot or in a vehicle couldn't reach or would be noticed at — over a fence line, above a facility, near a rooftop-mounted sensor.
- It converts a "would require breaching physical security" attack into a "line of sight from the property line or the air" attack — meaningfully lowering the cost/risk to the attacker for short-range protocols.
- Good discussion extension: does standard perimeter security (fences, cameras, guards) address this threat model at all? (Mostly no — most physical security is designed around ground-level intrusion, not aerial RF proximity.)

---

## 20. Should an organization simply ban wireless from safety-critical OT environments, or is controlled wireless with compensating controls a legitimate approach? What does a real wireless policy for OT need to address?

**Instructor Reference:**
- The "ban it all" argument: fewer failure modes, no RF attack surface at all, simplest to audit and enforce.
- The "allow with controls" argument: wireless retrofits are often far cheaper and faster than running new cable through an operating plant, and outright bans tend to produce unmanaged shadow-IT wireless deployments that are *less* visible and *less* controlled than a sanctioned, monitored one.
- A real policy needs to address: approved protocols/frequency bands, mandatory segmentation from safety-critical zones, key/credential management and rotation, a decommissioning process for retired wireless devices, and periodic RF surveys to catch unauthorized deployments.
- Tie back to NERC CIP-005 (Electronic Security Perimeters) if the class has covered NERC CIP — wireless access points are explicitly a documented entry point that must be accounted for in ESP definitions for BES Cyber Systems.
