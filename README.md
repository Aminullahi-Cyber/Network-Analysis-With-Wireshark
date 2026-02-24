# Network-Analysis-With-Wireshark

## Objective

The objective of this project was to perform a comprehensive forensic investigation of a 15-minute network packet capture (88,000 packets) to identify and document a full-spectrum web intrusion. By analyzing protocol hierarchies and TCP streams, I aimed to reconstruct an attack timeline that reduced the time-to-detection for a complex multi-stage threat including directory brute-forcing, SQL injection, and webshell deployment. The goal was to achieve 100% visibility into the adversary's "hands-on-keyboard" activity, ultimately identifying the specific vulnerabilities (insecure file uploads and SQLi) and the unauthorized egress callback on Port 4422 to provide data-driven mitigation strategies.

### Skills Learned

- Signature & Tool Identification
- Traffic Pattern Recognition & Basclining.
- Web Attack Forensics.
- Reverse Shell Analysis.
- Adversarial Mindset & Mitigation Logic.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- NetworkMiner
- Network analysis tools (such as Wireshark) for capturing and examining network traffic.
- URL decoder.
- TCPdump

## Steps

Step 1: I opened the PCAP file and started investigating using Wireshark.

<img width="960" height="507" alt="Screenshot 2026-02-04 031640" src="https://github.com/user-attachments/assets/e5e93ef1-0ea9-41fc-843f-eed3a2f21c5e" />

Step 2: To see what I can determine from the PCAP alone, I first open the capture file properties. Here I get alot of information about the PCAP.

<img width="955" height="505" alt="Screenshot 2026-02-04 031721" src="https://github.com/user-attachments/assets/0be46ce8-c3a2-4fc5-8adc-424be63d0e8c" />

Step 3: Next I looked carefully at the protocol hierachy where I had alot of protocols like SSH, DNS, HTTPS & SMB. I see this as potential lateral movement opportunities along with clear text protocols meaning I can read the exchange between HTTPS protocols and see what is going on.

<img width="956" height="502" alt="Screenshot 2026-02-04 031941" src="https://github.com/user-attachments/assets/e5d5fe22-e5d3-4876-b8ba-6ebaf842970f" />

Step 4: Next I moved to conversations and then to IPV4 and then to bytes. After that I moved to the TCP tables all while trying to look for useful information that will aid in this investigation.

<img width="957" height="487" alt="Screenshot 2026-02-04 032622" src="https://github.com/user-attachments/assets/2c043860-3f15-4217-800d-6d758214f361" />

<img width="960" height="502" alt="Screenshot 2026-02-04 033703" src="https://github.com/user-attachments/assets/a50083c7-f9a1-440e-8153-dbe8c7aa9939" />

Step 5: I then noticed that port 80 has a different byte (184 bytes) from the other ports. I also noticed that port 22 has 184 bytes, likely this means that the destination host has responded with SYN/ACK. SYN/ACK flag means the sender is acknowledging receipt of a client's SYN. The others did not respond to that because they have ports closed. So I can infer that IP address 10.251.96.5 has the port 80 & 22 opened.

<img width="960" height="502" alt="Screenshot 2026-02-04 033703" src="https://github.com/user-attachments/assets/383a5825-f6be-46d2-96ba-d33e58018a07" />


Step 6: Then also moving down, I noticed that IP address 10.251.96.4 with port 4442.

<img width="960" height="502" alt="Screenshot 2026-02-04 033703" src="https://github.com/user-attachments/assets/b7aa15b4-6037-4ba1-9fd8-c55ab4a6e0c2" />

Step 7: After analyzing the TCP tab, I then moved to UDP.

<img width="958" height="499" alt="Screenshot 2026-02-04 033923" src="https://github.com/user-attachments/assets/ed502bbe-d301-45d0-b973-8c043a4fbeb6" />

Step 8: This is where I start digging for gold. But before I start checking the PCAP, I moved to change the time on wireshark. I changed it to "UTC, date time and day format".

<img width="955" height="451" alt="Screenshot 2026-02-04 034147" src="https://github.com/user-attachments/assets/0edbec6e-52b5-4729-8207-2b1e42183cc4" />

Step 9: Beginning, I see a HTTP traffic on packet no 14. I right clicked on it, head over to follow & then TCP stream.

<img width="960" height="526" alt="Screenshot 2026-02-04 034931" src="https://github.com/user-attachments/assets/ad3dcb4d-e293-4bfe-be02-0368402ec835" />

<img width="945" height="525" alt="Screenshot 2026-02-04 034958" src="https://github.com/user-attachments/assets/e5d05690-974d-4175-8e28-10f33593e974" />

Step 10: This shows a communication between client and server.

<img width="954" height="540" alt="Screenshot 2026-02-04 035112" src="https://github.com/user-attachments/assets/257dbee3-19ab-492b-a783-ba28d4006afe" />

Step 11: Further scrolling down, I see a POST REQUEST on packet 38. Curious about what was entered in the post request in the packet, I decided to check the HTTP stream. 

<img width="955" height="488" alt="Screenshot 2026-02-04 120436" src="https://github.com/user-attachments/assets/9914437d-ec1d-4cc6-aa57-d0b5dda41a89" />

Step 12: Here I found a login with username & password as Admin%40.1234. Just as a reminder, special characters are encoded.

<img width="957" height="510" alt="Screenshot 2026-02-04 120524" src="https://github.com/user-attachments/assets/3063acc0-d084-461d-87fd-8e917217beaf" />

Step 13: And then out of curiosity, I googled URL decoder and paste the encoded character to get the output which turned out to be the "@" character. 

<img width="957" height="474" alt="Screenshot 2026-02-04 121031" src="https://github.com/user-attachments/assets/682789e3-fffb-4558-a342-9901c0b2c11e" />

Step 14: Before further analyzing the traffic, I decided to include both source ports and destination port into the packet list view.

<img width="960" height="492" alt="Screenshot 2026-02-04 121708" src="https://github.com/user-attachments/assets/c02f627a-0854-4031-800e-6ce833873d74" />

Step 15: Instead of manually scrolling, I decided to craft some display filters to zone off to the good stuff. I then move to filter to ease my analysis. Since I am interested in looking for 200 (which means the ones the web server replied to), I want to see all the 200s coming from the source 10.251.96.5.

<img width="943" height="100" alt="Screenshot 2026-02-04 124016" src="https://github.com/user-attachments/assets/4b7c1eb1-c7a3-4da4-8f9d-f0e30f0300e8" />

Step 16: After applying the filter, I got all the 200. But then my focus was on the length column. Most of the packets have a length between 500-750, but packet 7725 with length of 8494 which could only mean the server might have responded to something large.

<img width="956" height="494" alt="Screenshot 2026-02-04 124048" src="https://github.com/user-attachments/assets/a3beea58-6784-4074-838f-506a39749434" />

<img width="960" height="492" alt="Screenshot 2026-02-04 124645" src="https://github.com/user-attachments/assets/6c9c50b2-aaf5-4382-9c8c-a48b6d6ac041" />

Step 17: To confirm that, I hit the HTTP stream of the packet 7725 with length of 8494 to see more details. And Yes, we found alot of responses. I also noticed version of php, knowing the version, this means an attacker can look for exploit & exploit this particular version.

<img width="955" height="500" alt="Screenshot 2026-02-04 125340" src="https://github.com/user-attachments/assets/4c835402-d794-4344-9a77-94173fe9f7ec" />

<img width="960" height="508" alt="Screenshot 2026-02-04 125356" src="https://github.com/user-attachments/assets/91c8ad93-3ef0-473e-9265-175d0e93f459" />

Step 18: Going forward on Packet 14060, I noticed a wierd POST request. I then decided to follow that and check the HTTP stream.  

<img width="959" height="491" alt="Screenshot 2026-02-04 130253" src="https://github.com/user-attachments/assets/7e331dbc-a06b-4614-84e0-3967ac3846ce" />

Step 19: Upon checking the HTTP stream, there is clearly a form of SQL attack happening. But everything is encoded. I found out that it is indeed a form of SQL attack as well as testing to see if there are cross scripting.

<img width="960" height="489" alt="Screenshot 2026-02-04 130444" src="https://github.com/user-attachments/assets/29c0d2aa-80e6-4afb-b990-e278e3c99d29" />

<img width="689" height="475" alt="Screenshot 2026-02-04 130605" src="https://github.com/user-attachments/assets/f50a746f-ee74-4060-9d16-198224b979d0" />


### MITRE ATTACK Maping
- Reconnaissance: T1595 (Active Scanning)
The first red flag in the PCAP was the high volume of SYN packets.

The Evidence: Analyzing the Conversation Statistics in Wireshark showed a single source IP hitting over 1,000 destination ports in a matter of seconds.

Analyst Insight: This confirmed the adversary was mapping the target's "attack surface" before attempting an exploit.

- Persistence: T1505.003 (Web Shell)
Once the adversary found a vulnerability, they needed a way to stay in the system.

The Evidence: I used Follow HTTP Stream on the POST request to edit_profile.php and observed a multipart/form-data upload containing PHP code.

The Payload: The file db_functions.php acted as a "doorway," allowing the attacker to execute shell commands remotely via a browser.

- C2: T1571 (Non-Standard Port)
The most critical part of the investigation was the "callback."

The Evidence: I filtered for tcp.port == 4422 and found a successful three-way handshake followed by PSH (Push) packets containing cleartext Linux commands.

The Mitigation: This highlights a failure in Egress Filtering. Had the firewall been configured to block all outbound traffic except on known ports (like 80, 443, or 53), the reverse shell would have failed.





























































