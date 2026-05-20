# Detection bypass and defense tuning report Integrating Suricata with Splunk 

# Objective: 
The scenario focuses on bypassing security controls using Nmap evasion techniques and identifying potential logging and monitoring weaknesses within the target environment.

---

# Tasks: 

- perform a reconnaissance using nmap scan 
- Identify weaknesses
- Monitor the scan using suricata IDS in real time to detect threats and security gap detection
- Analyze alerts and correlate findings with SIEM (Splunk) logs 
- Tune detection rules and improve monitoring effectiveness   
- Propose defense-in-depth improvements and remediation actions  
---

# Introduction:
This report represents a complete purple team project in which an attacker attempts to bypass detection control by conducting reconnaissance using various evasion techniques. In the other side, the defender is responsible for detecting the evasion techniques and proposing tuning measures to improve detection effectiveness.

---

# Environment:
- **Attacker:** Kali virtual machine  
- **Target:** VMware Network `192.168.217.0/24` with 2 running machines (Kali and Ubuntu)  
- **IDS:** Suricata installed on Ubuntu server  
- **SIEM:** Splunk installed on Ubuntu server 

---

# 1. Bypass security control and identify weaknesses:

Attackers may attempt to bypass security controls by using various techniques such evasion detection and avoidance timing detection to identify vulnerabilities.

In this scenario, to fine tune the scan I used some flags to evade detection and reduce network load:
- **--data-length**: adds random data to the payload of packets, this increases the packet size by the specified number of bytes.  
- **--source-port**: use a specific port to appear legitimate.  
- **--min-parallelism**: the minimum number of parallel tasks launched to probe the target.  
- **-mtu 24**: to control the packets size to help fine tune the fragmentation.  
- **-n**: to skip DNS resolution.  
- **-Pn**: to skip host discovery.  
- **-D RND 7**: launch a scan using decoy IP addresses to confuse firewall and IDS, making it difficult to determine which IP initiates the scan.  
- **-sS**: SYN scan. It does not complete three-way handshake.  
- **-T1**: sneaky timing used to evade IDS detection and stay quiet in the network.  

---

# 1.1 Attack strategy:

1.1.1 IPv6 scanning  

<img width="943" height="361" alt="image" src="https://github.com/user-attachments/assets/7f9ab57f-2d14-45ae-bfc1-5c660cb4e62b" />


1.1.2 Full port sweep  

<img width="945" height="484" alt="image" src="https://github.com/user-attachments/assets/a4e61788-9e6b-4912-97fb-48b04acfc88e" />
 

1.1.3 NSE safe enumeration 

<img width="945" height="459" alt="image" src="https://github.com/user-attachments/assets/27dfb1e4-212a-4fc4-9766-ea5e5355ad23" />


1.1.4 Vuln script

<img width="945" height="455" alt="image" src="https://github.com/user-attachments/assets/325d1f2e-a4c1-4d0b-9fd2-3ca712dde899" />
 

1.1.5 Combined evasions 

<img width="944" height="625" alt="image" src="https://github.com/user-attachments/assets/a7791bf6-a3e3-4129-a4f2-8c4d417c600d" />
  

---

# 1.2 Findings:

The scan results in:

• IPv6 Blind spots which identifies exposed services running in IPv6 such as SSH, HTTP, memcahe services and others.  
• Hidden services: Indicates that many services are running behind the firewall in the network.  
• Weak firewall logging: Investigation of logs shows the absence of firewall logs. This is because various firewall evasions techniques were used such as MTU, --source-port, data-length and decoys.  

---

# 1.3 Attack mapping:

1. T1046: Network Service Discovery  
The attacker performed multiple scan techniques including full port sweep scan, vulnerability scan, service version detection and combined evasions techniques, to identify the services running on the remote host. This activity maps the MITRE ATT&CK technique T1046(Network Service Discovery) where adversaries attempt to discover networks services for exploitation.  

2. T1018: Remote System Discovery  
The attacker scanned the whole network range 192.168.217.0/24 to identify other systems and remote hosts within the network. This activity maps MITRE ATT&CK technique T1018(Remote System Discovery), in which the attacker attempts to get a listing of other systems on a network that may be used for lateral movement.  

3. T1595: Active Scanning  
The attacker performed active reconnaissance using Nmap to gather information that may be used during targeting. This activity maps the MITRE ATT&CK technique T1595 (Active Scanning), in which the attacker probes the victim to discover attack services.  

---

# 1.4 Why hardest:

This phase is the most challenging because it requires:

- Understanding logging architecture: the attacker must have a better understanding of logs and their locations such as host logs, firewall logs and IDS logs.  
- Blending multiple evasion techniques: the attacker relies on evasion techniques to blend in with normal activity, remain undetected and prolong their presence within the environment, to simulate realistic adversarial behavior.  
- IPv6 reconnaissance: The attacker must understand IPv6, the latest version of Internet Protocol, which was developed to address the limited address space of IPv4, contains 128 bits, represented as 8 groups with 4 Hexadecimal digits separated by colons.  

---

# 2 Detection analysis:

By enabling Suricata real-time network monitoring and correlating the generated alerts with Splunk, several potential threats associated with Nmap scan were identified:

• IPV4 fragmentation overlap detection  
• SSH and VNS scan detection  
• Excessive retransmission detection  
• ACK anomalies detection  
• HTTP spoofing port detection  
• Suspicious inbound mySQL scan detection  

Monitoring was performed using the following command:  

<img width="945" height="74" alt="image" src="https://github.com/user-attachments/assets/92a639a6-a77b-4ee4-9432-9fd9624c9a5b" />


---

# 2.1 Security gap detection:

# 2.1.1 IPV6 monitoring verification:

The SPL Splunk result confirmed that IPV6 traffic was successfully monitored by Suricata.  

<img width="945" height="517" alt="image" src="https://github.com/user-attachments/assets/7483dac5-d663-4e4b-a9c2-415fb7db82a3" />

# 2.1.2 Fragment reassembly check: 

The SPL query in Splunk indicated that fragments were successfully reassembled.  

<img width="945" height="533" alt="image" src="https://github.com/user-attachments/assets/496bf1cd-bf98-4c5b-b2e4-38078a86da8b" />


---

# 3 Remediation:

# 3.1. Suricata rule tuning: 
To improve the Suricata detection, I created custom rules in a file named local.rules and configuring Suricata to load them by adding the path file to /etc/suricata/suricata.yaml  

<img width="945" height="457" alt="image" src="https://github.com/user-attachments/assets/64680eff-35c3-4a9c-8b01-d97e772e94f9" />

# 3.1.1. Unusual port sources detection: 
The following rule is created to detect and alert on unusual port sources.

<img width="945" height="467" alt="image" src="https://github.com/user-attachments/assets/1a381e25-d6fd-4a8b-a206-2e25a2a59dcc" />

# Rule validation:
Next, I executed the following command to test the created rule by performing a scan using a spoofed and trusted port source  

<img width="943" height="156" alt="image" src="https://github.com/user-attachments/assets/92b704a7-0b9e-4c50-a0fc-03d6141757dd" />

# Generated alert:
As a result, Suricata successfully detected and generated an alert on unusual port source.

<img width="944" height="85" alt="image" src="https://github.com/user-attachments/assets/891df3e9-e291-49ac-aa7f-29b111bcd678" />

# 3.1.2. Low and slow scan detection: 
The following rule is created to detect and alert on low and slow scan.

<img width="945" height="439" alt="image" src="https://github.com/user-attachments/assets/e5dcdd94-52fc-496d-bcec-ad7c39a18e0a" />

# Rule validation:
Next, I executed the following command to test the created rule by performing a scan using sneaky timing:

<img width="944" height="79" alt="image" src="https://github.com/user-attachments/assets/b09cb196-3515-4a26-8ccb-135776cd493f" />

# Generated alert:
As a result, Suricata successfully detected the slow scan and generated an alert.

<img width="943" height="149" alt="image" src="https://github.com/user-attachments/assets/ad6d4b4c-da84-4a24-954e-d6c1fe4a9c2d" />

# 3.1.3. Internal scan detection:
The following rule is created to detect and alert on internal scan.

<img width="945" height="468" alt="image" src="https://github.com/user-attachments/assets/d803cd01-66ed-40c7-9198-9efd22d02d7c" />

# Rule validation:
Next, I executed the following command to test the created rule by performing a stealthy scan on an internal host:

<img width="943" height="163" alt="image" src="https://github.com/user-attachments/assets/9a4cec7e-ec00-47bb-80c4-1303b3647886" />

# Generated alert:
As a result, Suricata successfully detected and generated an alert on internal scanning.

<img width="944" height="67" alt="image" src="https://github.com/user-attachments/assets/1325b994-5962-4482-b54c-00ea98662974" />


---

# 3.2 Firewall rule tuning:

I configured iptables to allow only required MAC address and deny all others MAC addresses by default  

<img width="944" height="124" alt="image" src="https://github.com/user-attachments/assets/8f21ca44-a1cd-4aa7-896b-7412182edb41" />

# 3.2.1. Filtering Mac address using iptables:
The following commands were used to spoof the original mac address  

<img width="943" height="41" alt="image" src="https://github.com/user-attachments/assets/463850fd-2b64-4744-8f10-83d4bc42b093" />

# Rules validation:
I executed the following command to test the firewall rule:

<img width="944" height="96" alt="image" src="https://github.com/user-attachments/assets/39276f97-6905-40bc-8d01-33c231489c40" />

 # Generated alert:
As a result, firewall log was successfully enabled.

<img width="943" height="133" alt="image" src="https://github.com/user-attachments/assets/ef8561d7-3e96-4cce-ac6c-d97676353e93" />

 

---

# 4 Recommendations:

• Forward firewall logs to the Splunk SIEM to enhance visibility and detection capabilities.  
• Configure Suricata in IPS mode for deep packet inspection and blocking malicious traffic rather than just relying on signature.  
• Disable unused ports to reduce the attack surface.  
• Segment the network using VLAN.  
• Implement egress filtering to restrict outbound traffic to known malicious IP or domain.  

---

# Conclusion:
During this scenario, integrating Suricata with Splunk, tuning Suricata and firewall rules, and providing a remediation plan helped enhance defenses to effectively detect evasion techniques used to bypass security controls.
