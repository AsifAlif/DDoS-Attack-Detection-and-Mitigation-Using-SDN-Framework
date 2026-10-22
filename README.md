DDoS Attack Detection and Mitigation Using an SDN Framework
Real-time detection and mitigation of DDoS attacks in a software-defined network. The system uses the Ryu OpenFlow controller together with sFlow-RT telemetry to identify volumetric flooding attacks as they happen and automatically block the malicious traffic at the switch, keeping the network available for legitimate users.
Demo video: https://drive.google.com/file/d/1jLFl_DEF1h1SK4ANQu-8rMBZ5TCe25xR/view?usp=sharing
---
What this project does
A Distributed Denial-of-Service (DDoS) attack floods a target with traffic until legitimate users can no longer reach it. This project defends against such attacks using Software-Defined Networking (SDN), where a central, programmable controller can see traffic across the whole network and push mitigation rules to the switches the instant an attack is detected.
The project implements the full defense pipeline on an emulated SDN:
Traffic monitoring. Each switch continuously exports sFlow samples of the traffic it forwards to sFlow-RT, which maintains a live view of the largest flows in the network.
Attack detection. A detection script running inside sFlow-RT evaluates these flows against a bandwidth threshold. When a single flow's rate crosses that threshold — the signature of a volumetric flood — it raises an alert.
Automatic mitigation. On that alert, the script calls the Ryu controller's REST API to install an OpenFlow rule that drops the malicious flow directly at the switch. The flood is stopped in the data plane, so it never reaches the victim and the controller is not overloaded.
Live visualization. The sFlow-RT dashboard shows the top flows and port utilization in real time, so the attack spike and its mitigation are clearly visible as they occur.
The result is an automated system that detects and blocks DDoS traffic within seconds, with no manual intervention.
---
Technologies used
Ryu — OpenFlow SDN controller (`simple_switch` for Layer-2 forwarding and `ofctl_rest` for programmatic flow-rule installation).
Mininet — emulates the software-defined network: Open vSwitch switches, hosts, and bandwidth-limited links.
sFlow-RT — real-time flow analytics engine that ingests telemetry and runs the detection and mitigation logic.
OpenFlow 1.3 — protocol used by the controller to install the drop rules on the switches.
Kali Linux — the environment hosting the complete stack.
Emulated network: a tree topology (depth 2, fanout 2) of three OpenFlow switches and four hosts, with every link limited to 10 Mbit/s so that a flood visibly saturates the path and the effect of mitigation is clear.
---
How to run
Run each step in its own terminal.
1. Start the Ryu controller with Layer-2 forwarding and the REST API:
```bash
ryu-manager ryu.app.simple_switch ryu.app.ofctl_rest
```
2. Start sFlow-RT with the detection and mitigation script:
```bash
./sflow-rt/start.sh -Dscript.file=./ryu.js
```
sFlow-RT receives telemetry on UDP port 6343 and serves its API and dashboard on port 8008.
3. Build the network in Mininet, enabling sFlow export and connecting the switches to the controller:
```bash
sudo mn --custom sflow-rt/extras/sflow.py \
        --link tc,bw=10 \
        --controller=remote,ip=127.0.0.1 \
        --topo tree,depth=2,fanout=2
```
4. Open the dashboard in a browser:
```
http://127.0.0.1:8008/app/mininet-dashboard/html/index.html
```
5. Test it. Confirm connectivity with `pingall`, then launch a DDoS flood from one host against the victim. On the dashboard the attack appears as a sharp spike in the top flows and saturation of the switch ports; moments later the mitigation rule drops the malicious flow, the rate falls back, and legitimate traffic continues unaffected.
---
Results
Detects volumetric DDoS floods in real time from live network telemetry.
Mitigates the attack automatically by dropping the malicious flow at the switch within seconds of detection.
Keeps the control plane lightweight, because enforcement happens in the data plane rather than by routing attack traffic through the controller.
Makes the attack and its mitigation clearly observable through the live dashboard.
---
Key concepts demonstrated
Software-Defined Networking and the OpenFlow control-plane / data-plane separation
Telemetry-driven, threshold-based DDoS detection with sFlow-RT
Automated mitigation through the Ryu REST API and OpenFlow flow rules
SDN emulation and traffic engineering in Mininet (custom topology and bandwidth limits)
Author
Asif Alif — network security, SDN, and post-quantum cryptography.
