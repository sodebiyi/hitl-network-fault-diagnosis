# A Human-in-the-Loop AI System for Network Fault Diagnosis and Remediation

Olusegun Victor Odebiyi

MSc Advanced Computer Networks, Sheffield Hallam University

## Abstract

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
