# Hive — Plan

Oct 19, 2026 · @Asish

## Vision and positioning

Build Hive (Hands-on IoT Virtual Environment), a fully in-browser platform where people experience building complete IoT systems — edge node, network, storage, web app, security and ML — with no hardware and no installs. It is a sandbox with starting worlds, not a lab course: faculty bring their own curriculum and do their own grading.

- **The problem:** individual IoT hardware is expensive for students and universities, and setup (Docker, toolchains) blocks second-year students.
- **The gap:** Wokwi and Tinkercad simulate single microcontrollers; Cisco Packet Tracer covers networking; Node-RED covers flows. None gives a system-level, end-to-end IoT experience with realistic faults and attacks in a browser tab.
- **Positioning:** "experience IoT systems," not "virtual Arduino." The professor's course is the first and best-supported use, not the only one.
- **Rule:** reuse open source wherever it exists; build only the glue, the broker and the world engine.

## Users and the professor's curriculum

First users are second-year ECE and CS students who may not be able to run Docker, so everything runs in a browser. Kids come later through a drag-and-drop layer (Node-RED-style or Blockly) on the same engine.

- **Students:** build and run systems, share world snapshots as submissions.
- **Faculty:** create scenarios (a starting world, a brief, starter code), share links, grade submitted snapshots their own way.
- **Self-learners and other universities:** use the public scenario library.

The professor's four labs become four scenarios she authors, each layering onto the previous one:

| Lab | Scenario focus | What students do in the browser |
| --- | --- | --- |
| 1. Edge node bring-up, sensing and actuation | Virtual Raspberry Pi with sensors and actuators | Write gpiozero-style Python against virtual pins; see the board react (polling vs interrupts, debouncing, sampling) |
| 2. MQTT networking and a data pipeline to a database | In-browser broker + PGlite | Publish from the edge node, build an ingest path to Postgres, learn QoS, retained messages, Last Will under injected latency and loss |
| 3. Full-stack IoT with web-based control | Dashboard + fake REST layer | Build a control page; commands flow back down to the virtual actuators |
| 4. Cybersecurity and machine learning | Attack scenarios + labeled data | Watch traffic, survive spoofing and replay attacks, fix with HMAC, encryption and ACLs; train an anomaly detector on labeled data |

## Architecture and stack

All student work runs in the browser tab; one small PocketBase server adds accounts, snapshot storage and sharing. Static hosting needs COOP/COEP headers (Cloudflare Pages or Netlify, not GitHub Pages).

&#91;embedded content: platform architecture · browser tab plus one small server\]

The world engine and broker are the parts we build; everything else is reused open source.

| Layer | Component | License | Notes |
| --- | --- | --- | --- |
| Editor | Monaco | MIT | The editor behind VS Code |
| Student code runtime | Pyodide in a Web Worker | MPL 2.0 | Real Python incl. scikit-learn; Stop button via SharedArrayBuffer |
| Virtual board visuals | wokwi-elements | MIT | LEDs, sensors, displays as web components |
| Arduino-style firmware (optional) | avr8js | MIT | Runs compiled Arduino sketches; avoid Wokwi's hosted ESP32 simulator (not open source) |
| Broker | Own TypeScript MQTT broker | Ours (Apache 2.0) | Deterministic, packet hooks for the traffic inspector, fault injection; core IP |
| Database | PGlite (Postgres in WebAssembly) | Apache 2.0 | Persisted to IndexedDB; real SQL |
| Fake backend | Mock Service Worker | MIT | Student pages make real fetch calls |
| Charts | Chart.js | MIT | Dashboards |
| Crypto | WebCrypto | Browser API | Real HMAC signing and payload encryption |
| App shell | React or Svelte + TypeScript + Vite | MIT | Fixed-timestep world engine |
| Tests | Playwright | Apache 2.0 | End-to-end tests Claude can run |
| Server | PocketBase (single Go binary) | MIT | Auth, snapshots, sharing, scenario library |
| Real broker (optional, later) | Mosquitto over secure WebSockets via MQTT.js | EPL/EDL, MIT | For real TLS and credentials; run as a separate service |

Licensing rule: bundle MIT, Apache and BSD freely with notices kept; run EPL and GPL tools as separate services, never linked. Re-check every license before publishing.

## Experience design

There is no "complete" state: the achievement is the system you build, and the world keeps throwing events at it.

- **Persistent worlds:** a smart home and a small factory first; a greenhouse or city block later. Devices live in the world and react visibly when code runs.
- **Events:** heatwave, a door opening at 3am, a pump failing, packet loss, an attacker on the network. Systems must handle what happens rather than pass a test.
- **Growth:** start with one sensor; end with edge node, broker, database, dashboard, security layer and ML. A live architecture map grows as pieces are added.
- **Sharing:** every world is a snapshot link (code included) — used for submissions, grading and showing off.
- **Scenarios:** a starting world + a short brief + optional starter code, authored by faculty. Start as JSON; add an editor UI later. A public scenario library is the long-term moat.
- **First 10 minutes:** a guided start in a default world, to beat the blank-canvas problem.
- **Project safety:** export and import from day one, since browser storage can be cleared.

## Scope

Three tiers, each a strict subset of the next; the 48-hour version is the recommended first build.

| Tier | Includes | Leaves out |
| --- | --- | --- |
| 12-hour hackathon | Monaco + Pyodide worker, one world with temperature, motion, LED and fan, in-browser broker with topics and wildcards, Lab 1 experience end to end | Database, dashboard, security, ML, accounts |
| 48-hour hackathon | All of 12h + PGlite ingest and SQL console, dashboard and control page, traffic inspector with one spoofing attack and HMAC fix, labeled anomaly toggle with Isolation Forest, export/import | Accounts, sharing, scenario editor, multiple worlds |
| Full MVP | All of 48h + PocketBase accounts, snapshot sharing, scenario format and library, two worlds (home, factory), event scheduler, full MQTT behaviour (QoS 0/1/2, Last Will, sessions), five or six attacks, architecture map, accessibility, Chrome/Firefox/Edge support | Kids' drag-and-drop, real-hardware bridge, LMS/LTI, UVic SSO, autograding |

**48-hour plan**

1. 0–4h: Vite shell, Monaco, Pyodide worker with Stop, deployed to Netlify with COOP/COEP headers
2. 4–10h: world engine, virtual board, gpiozero-style Python API
3. 10–16h: in-browser broker (topics, wildcards, retained) and Python publish/subscribe API
4. 16–20h: edge-node experience working end to end
5. 20–26h: PGlite storage, topic-to-table ingest, SQL console
6. 26–32h: Chart.js dashboard and control page driving actuators
7. 32–38h: traffic inspector, spoofed-sensor attack, HMAC fix
8. 38–42h: anomaly toggle with labels, Isolation Forest cell
9. 42–46h: export/import, polish
10. 46–48h: demo script, recording, README

Cut first if behind: ML, then the fake REST layer (the control page can talk to the broker directly).

Demo story: one tab, no installs — write code, light an LED, stream to Postgres, control it from a web page, get attacked, fix it, catch the attack with ML.

## Challenges and risks

The hardest technical risks are in the browser runtime; the hardest product risks are adoption and the blank canvas.

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Pyodide + scikit-learn first load is tens of MB | Slow start, low-RAM Chromebooks struggle | Service-worker cache; lazy-load scikit-learn only when ML is used |
| Student code blocks or loops forever | Frozen tab | Run in a Web Worker with a Stop button; SharedArrayBuffer for interrupts and real sleep (needs COOP/COEP headers) |
| No real ports, sockets or TLS handshakes | Teaches MQTT semantics, not broker operations | Optional "connect to a real broker" toggle via MQTT.js |
| TLS can't be shown honestly in-browser | Weaker security lessons | Real application-layer crypto via WebCrypto; real TLS through the optional broker |
| Browser storage can be cleared | Lost work | Export/import from day one; PocketBase snapshots in the full MVP |
| Safari quirks with workers and storage | Bugs on Macs and iPads | Officially support Chrome, Firefox and Edge |
| Blank-canvas problem | Students stall | Default worlds and a guided first 10 minutes |
| Faculty adoption | Few users beyond one professor | Ship her four scenarios polished as proof |
| Free sandbox is hard to monetize | No business | Institutional tier: private class spaces, custom worlds, support |
| Scope creep | Never ships | Hold the tier boundaries above |

## Effort and build approach

With heavy Claude Code (Max 20x) use, the full sandbox MVP is about 3–5 weeks full-time, or one semester at 10–15 hours a week; the binding constraint is review and design time, not code.

| Phase | Estimate |
| --- | --- |
| Specs and architecture (world model, broker interface, event schema) | 2–3 days |
| Core platform | 1.5–2.5 weeks |
| Scenarios and first-run experience | about 1 week |
| Pilot with students and fixes | about 1 week |

- **Speeds up a lot:** the broker (test-first against MQTT behaviour), integration glue, UI scaffolding, tests.
- **Barely speeds up:** experience design, browser edge cases, simulation realism choices, scenario content, piloting.
- **How to work:** a CLAUDE.md with architecture rules; parallel sessions per module (broker, world engine, Python runtime, backend, UI), each in its own worktree with an interface spec; deterministic fixed-timestep engine so tests verify behaviour; Playwright so Claude checks its own UI.

## MASc fit, IP and business model

The platform is the research instrument, not the thesis; it fits the proposed topic of a reconfigurable edge-to-cloud testbed for secure, resilient cyber-physical systems.

1. **Virtual half of a hybrid testbed (backbone):** virtual nodes and real ones (Raspberry Pi, F1TENTH) share one broker. Question: how faithfully can simulated nodes stand in for hardware in resilience and security experiments, and where do they break?
2. **Browser as a commodity edge device (early experiment):** measure latency, throughput, ML inference and energy of WebAssembly runtimes on phones and low-spec laptops against a Raspberry Pi.
3. **Engineering education study (side paper):** learning outcomes vs hardware labs; venues such as IEEE EDUCON or Frontiers in Education; needs UVic research ethics approval.

**First paper:** a system paper on a browser-based reconfigurable IoT/CPS testbed for IEEE CCECE 2027 (full paper due January 25, 2027), with one small evaluation. Needs the platform core by late December.

**IP and open source:** thesis work falls under UVic's IP policy and any funding agreements; check the policy and agree terms in writing with the professor before building. Proposed model is open core: engine under Apache 2.0; hosted classroom version, curated worlds and support kept as the product.

**Business model:** free for students and self-learners; paid institutional tier for private class spaces, custom worlds and support.

## Next steps

- [ ] Agree IP ownership and open-source terms with the professor, checked against UVic's IP policy
- [ ] Confirm with her that the sandbox-with-scenarios model replaces graded labs, and who writes the four scenarios
- [ ] Write the specs: world model, broker interface, event schema, scenario JSON format
- [ ] Set up the repo, CLAUDE.md and Netlify deploy with COOP/COEP headers
- [ ] Build the 48-hour version
- [ ] Decide whether to pursue the CCECE 2027 system paper
