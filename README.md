# Case Study:  Optimizing Vendor Connections through Source IP Anchoring

## Overview:
</br>

<img align="left" alt="Portfolio Logo" width="200px" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQyMk6xX2_L1CvEBpw6xu1ipeeYuMHeE8R6jg&s" />
</br>

- **Position**:  Information Security Analyst, Subject Matter Expert (SME)  
- **Timeframe**:  One month  
- **Objective**:  Optimize vendor connections while maintaining a strong security posture by utilizing source IP anchoring. 
</br>
</br>
</br>
</br>


## Background
- **Context**:  A user submitted a service ticket requesting that we whitelist a new vendor's domains in Zscaler ZIA.  It was assumed that Zscaler was blocking the connection.  
- **Stakeholders**:  InfoSec Team, New Vendor's Technical Team, Requesting Department, Compliance Team, Zscaler

## Problem Statement
- The organization needed to connect securely to a vendor's application but encountered a problem when a service ticket requested whitelisting of the vendor’s domains in Zscaler ZIA.  Despite no blocks in Zscaler logs, the vendor provided broad domain requests, prompting the need for a solution that adhered to security protocols

## Approach
- **Research and Analysis**:  Conducted a detailed analysis of the user's machine, the application, and Splunk logs, while reviewing Zscaler documentation.
- **Methodology**:  Worked closely with the requesting user and the vendor to clarify connection requirements and gather specific domain information.  Developed a systematic approach for testing the connection.  
- **Tools and Technologies**:  Utilized Zscaler Internet Access (ZIA) for traffic filtering and Zscaler Private Access (ZPA) for secure application access, analyzed connection issues through Splunk logs, analyzed PCAP files with Wireshark. 

## Implementation
**Steps Taken**:
1. Initial Investigation:  Checked Splunk logs for the vendor's domain and found no blocks by Zscaler; all connections were allowed.  The second domain had a wildcard: *.s3.amazonaws.com, which posed significant security risks by granting overly broad access to AWS services.
2. User Meeting:  Requested a meeting with the vendor to clarify specific domains.
3. Testing Session:  Scheduled a day for testing with the vendor and involved coworkers.  During the testing, the connection failed with Zscaler enabled but was successful when Zscaler was temporarily disabled.
4. Log Analysis:  Pulled up Splunk logs and identified that Zscaler blocked the connection due to a dropped SSL handshake.  Suspected the failed handshake and subsequent dropped connection was due to Zscaler's role as a man-in-the-middle, disrupting the pinning process required by the vendor's application.
5. Domain Whitelisting:  As a troubleshooting step, entered the first domain into ZIA's SSL Bypass list and re-enabled Zscaler.  This resolution would resolve both certificate and IP pinning issues.  The connection test was successful.
6. Ongoing Connection Issues:  After some time, another connection error occurred, this time linked to one of the AWS domains.  Added the long list of AWS domains to the SSL Bypass.  The connection was then successful.
7. Wireshark Analysis:  Removed all the domains from the SSL Bypass, then captured and analyzed packets to determine the failure point.  Wireshark revealed that the connection dropped due to IP pinning, not certificate pinning.  
8. IP Pinning:  In the PCAP file, Wireshark revealed that the connection began with one Zscaler proxy IP, but midway through the session, the IP switched to a different Zscaler proxy.  This violated the vendor's IP pinning requirement, leading to the termination of the connection, as seen by the RST (Reset) packets from the vendor server.  This confirmed the issue was due to IP pinning, not certificate pinning.
9. Introducing Source IP Anchoring:  Educated the vendor about Zscaler's ZPA and source IP anchoring and asked analyst to confirm with his firewall admins whether or not they required our public IP ranges for consistent communication with their application.
10. Final Implementation:  Removed previous SSL bypass entries, adding them to ZPA instead, preserving Zscaler’s traffic inspection while ensuring successful connections with the vendor.Added all vend
11. Testing Confirmation:  Conducted final tests on the application, which were successful.  The entire process, including necessary approvals and change controls, took about a month.

**Challenges Faced**:  The broad wildcard domain (*.s3.amazonaws.com) requested by the vendor posed a significant security risk, allowing unrestricted access to a wide range of AWS services.  Additionally, Zscaler's role as a man-in-the-middle disrupted the SSL handshake by interrupting the certificate pinning process, complicating the connection.  While implementing an SSL bypass for specific domains was suggested by Zscaler's documentation, it primarily addressed Zscaler Internet Access (ZIA) and did not account for Zscaler Private Access (ZPA).  Coordinating with the vendor to provide more specific domains and adjusting the configuration to utilize Source IP Anchoring further ensured a secure connection without compromising security protocols.

## Results
- **Outcomes**:  Successfully implemented Source IP Anchoring for the vendor's application, ensuring secure and seamless connections while maintaining Zscaler traffic inspection.
- **Impact**:  Enhanced the organization's security posture by eliminating the need for an SSL bypass, adhering to NIST 800-53 Rev 5 guidelines.

## Lessons Learned
- **Departmental Education**:  Ensuring department heads and end-users are more aware of security protocols and potential solutions can help mitigate issues earlier in the process, streamlining troubleshooting and reducing time spent on reactive measures.
- **Proper Use of Zscaler Controls**:  Instead of temporarily disabling Zscaler, signing out of the tool during troubleshooting would reinforce proper usage and prevent end users from inadvertently learning how to bypass security controls.  This approach ensures continued adherence to security protocols without introducing unnecessary risks.

## Conclusion
- By prioritizing source IP anchoring over SSL bypassing, I better aligned our security practices with NIST SP 800-53 Rev 5 controls that advocate for robust access control, monitoring, and data protection.  This approach not only upheld our security posture but also demonstrated compliance with federal standards for information security.

### References
- [ZIA SSL Inspection Leading Practices Guide](https://help.zscaler.com/zscaler-deployments-operations/zia-ssl-inspection-leading-practices-guide)
- [Configuring Source IP Anchoring](https://help.zscaler.com/zia/configuring-source-ip-anchoring)
- [Certificate Pinning and SSL Inspection](https://help.zscaler.com/zia/certificate-pinning-and-ssl-inspection)
- [NIST SP 800-53 Rev. 5:  Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
    - #### Relevant NIST SP 800-53 Rev 5 Controls
      - ##### AC-17: Remote Access
          - This control emphasizes the importance of secure remote access, which was upheld by implementing source IP anchoring to ensure that only authorized IP ranges could access the vendor's services.
      - ##### SC-12: Cryptographic Key Establishment and Management
          - By avoiding broad SSL bypasses and maintaining secure communications, the analyst ensured that encryption was enforced, reducing the risk of data exposure.
      - ##### SC-28: Protection of Information at Rest and in Transmission
          - The use of source IP anchoring allowed for inspection of traffic to and from the vendor, preserving the integrity and confidentiality of sensitive information.
      - ##### SI-4: Information System Monitoring
          - By maintaining traffic inspection through Zscaler instead of bypassing it, the organization enhanced its capability to monitor for malicious activity and ensure compliance.
