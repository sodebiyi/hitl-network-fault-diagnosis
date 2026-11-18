# A Human-in-the-Loop AI System for Network Fault Diagnosis and Remediation

Olusegun Victor Odebiyi

MSc Advanced Computer Networks, Sheffield Hallam University

## PROJECT SCOPE:

During my MSc in Advanced Computer Networks at Sheffield Hallam, I built an AI-assisted system for network fault diagnosis that keeps the engineer in control. I used Claude to write the Python scripts for the system, used OpenAI GPT-4.0 as the LLM, and built the engineer interface in Streamlit. Most of the work was in the automation around the model, not the model itself.

I started by designing the system, then setting up the infrastructure in a virtualized environment, integrating the various nodes, injecting faults, testing and evaluating the outputs. I designed the flow first, log collection, source of truth, RAG, API to the LLM, and approval in Streamlit, so each part could be built and tested separately. I then built the lab in a virtualized environment using EVE-NG and CML, with Catalyst 9300 switches, SD-WAN 8200 routers and a Firepower firewall as virtual nodes, plus separate virtual machines for the Ubuntu rsyslog server, NetBox and the Streamlit app. I integrated the various nodes by configuring forwarding, SSH access and API access between them, so logs, topology and configs all flowed into one pipeline.

I started building the source of truth, because without it the AI had nothing reliable to work from. I used Claude to write the Python script using Netmiko to SSH to each device to automatically pull the config. The script logged into each device, ran show running-config, show ip interface brief, show cdp neighbors detail, show vlan brief and show ip ospf neighbor, parsed the output and pushed it to NetBox through the NetBox API, creating and updating devices, interfaces, IP addresses and cables. I ran it on a schedule and after any change, so NetBox stayed as the intended state and the config was always pulled automatically, not copied manually. I also kept intended configs in Git, and I wrote a difference engine that compared observed state against intended state and produced a structured difference. I reviewed, tested and debugged every script Claude drafted in the lab before I trusted it.

For logs, I set up a central rsyslog server on Ubuntu Linux. I configured each device to forward syslog to that server, and I configured rsyslog to write per-device files under /var/log/remote using templates based on hostname and facility. I used rsyslog queues so bursts did not drop messages when the pipeline was busy.

The RAG system was built to activate any time a log arrived at the server. A Python watcher, also written with Claude, tailed the rsyslog files, and on each new batch it triggered the RAG pipeline. The RAG step pulled relevant context, the device and topology from NetBox, the intended config from Git and past cases, and packaged that as feedback for the next step.

I connected the RAG to the LLM through an API. The Python code took that feedback and built the prompt for OpenAI GPT-4.0, then sent it through the OpenAI API and waited for the response. The prompt contained the log, the difference and the topology, and OpenAI GPT-4.0 had to return structured JSON with root cause, evidence, confidence, a plain English summary and remediation steps. I handled the API call in code with timeout, retry and error handling, so a failed call did not crash the pipeline. I validated the returned JSON against a schema, and if it failed or confidence was low, the result was flagged as low confidence in the UI. For remediation, I generated Ansible playbooks from templates and CLI commands as text, with a dry-run and syntax check so a bad playbook failed before it ever reached a device.

For the engineer to review it, I built the output view in Streamlit. The Streamlit app showed the alarm, the OpenAI GPT-4.0 diagnosis, the highlighted difference and the proposed playbook. The engineer could accept, edit or reject with a reason, directly in Streamlit. No change was sent to a device without approval in Streamlit. Every decision was logged with the difference and model output, and that feedback was fed back into the RAG retrieval so future cases had better context.

I automated the testing as well. I wrote a fault injection script that replayed the same twenty faults, configuration drift, link failure, resource exhaustion and multi-domain faults, so I could compare runs. I tested against manual log review as a baseline, using other MSc students as engineers. I logged whether they found the correct root cause, how long it took from alarm to a validated fix plan, whether they accepted an incorrect fix, and the precision of the system.

In testing, the system was able to localize the issue, provide RCA, reduce MTTR and increase productivity, and it produced 85.5% precision. It localized faults to the specific device and interface, instead of leaving the engineer to search through a burst of alarms. It provided root cause analysis with evidence and confidence, for example a VLAN mismatch where the code pointed to the exact field that differed, and a multi-domain fault where a transport power failure took down a wireless cell and the system linked the two events instead of treating them as separate alarms. Because engineers did not have to manually correlate logs, the mean time to repair from alarm to validated fix plan went down, and they could handle more faults in the same time. Where the evidence was thin, engineers overrode it, which was the behaviour I wanted.
Diagnosing faults in enterprise networks is still largely manual. Engineers have to work through large volumes of logs, compare what they see against what the network is supposed to look like, and decide on a fix, often under time pressure. Fully automated approaches promise speed, but many network teams are reluctant to hand remediation decisions to a system they cannot inspect or override.

This dissertation designs, implements and evaluates a Human-in-the-Loop (HITL) AI system for network fault diagnosis and remediation. The system does the slow analytical work, while the engineer keeps control of every remediation decision. It captures syslog events with an rsyslog server and correlates them against a baseline of topology and configuration held in a NetBox source of truth. The correlated context is passed through a Retrieval-Augmented Generation (RAG) pipeline to a large language model, which produces a diagnosis and recommended remediation steps for the engineer to review.

The work follows a Design Science Research approach within a positivist, deductive experimental framework. The system was tested in a virtualised enterprise LAN built in EVE-NG, made up of access and distribution switches and a core router running EIGRP. A set of EIGRP faults was injected, including interface shutdowns, passive-interface misconfiguration, AS-number mismatch, K-value mismatch, authentication mismatch, duplicate router IDs, duplicate network IDs, incorrect IP addressing and process restarts.

Faults were detected within 2 to 5 seconds, and a full diagnosis with remediation guidance was produced within 10 to 17 seconds. The root cause was localised correctly in nearly all cases. Across the evaluation there were 7 true positives, 1 false positive and no false negatives, giving a precision of 85.7%, a recall of 100% and an F1 score of 92.3%. The single false positive came from an EIGRP process fault being confused with background log noise.

The contribution is an integrated architectural blueprint and an empirically validated evaluation framework for trustworthy, explainable AI-assisted network fault management.

## How the system works

Network devices send syslog messages to an rsyslog server. When an event arrives, the system looks up the affected device and its expected configuration in NetBox, which acts as the source of truth. That context, together with the log evidence, is fed into a RAG pipeline so the language model reasons from the actual state of the network rather than from general knowledge alone. The model returns a likely root cause and suggested remediation. Nothing is changed on the network until the engineer reviews the recommendation and decides what to do.

## Test environment

The lab was built in EVE-NG as a virtualised enterprise LAN with access and distribution switches and a core router running EIGRP. Fault scenarios were injected one at a time so that detection time, diagnosis time and accuracy could be measured for each.

## Results

Detection took 2 to 5 seconds. Diagnosis with remediation guidance took 10 to 17 seconds. Root cause was identified in nearly all scenarios. The confusion matrix was 7 true positives, 1 false positive and 0 false negatives, which gives 85.7% precision, 100% recall and a 92.3% F1 score. The one false positive was an EIGRP process fault that the model confused with background log noise, which points to log filtering as a useful area for improvement.

## Future work

Three directions look most promising. The first is a dual source of truth, so the system can cross-check intended configuration against the live running state. The second is an active verification loop that tests a proposed change before it is committed. The third is bringing in other telemetry sources such as SNMP and NetFlow, so diagnosis does not depend on syslog alone.

## Keywords

Network fault diagnosis, Human-in-the-Loop, Retrieval-Augmented Generation, large language models, EIGRP, NetBox, EVE-NG, explainable AI, network automation
