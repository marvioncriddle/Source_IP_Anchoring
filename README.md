# Case Study:  Optimizing Vendor Connections through Zscaler ZIA

## Overview:
</br>

<img align="left" alt="Portfolio Logo" width="400px" src="https://i.imgur.com/YesIrJe.jpg" />
</br>

- **Position**:  Information Security Engineer, Subject Matter Expert (SME)  
- **Timeframe**:  One month  
- **Objective**:  Optimize vendor connections while maintaining a strong security posture by utilizing Zscaler, our Secure Web Gateway (SWG). 
</br>
</br>
</br>


## Background
- **Context**:  A user submitted a service ticket requesting that we whitelist a new vendor's domains in Zscaler ZIA.  It was assumed that Zscaler was blocking the connection.  
- **Stakeholders**:  InfoSec Team, New Vendor's Technical Team, Requesting Department, Compliance Team, Zscaler

## Problem Statement
- The organization needed to connect securely to a vendor's application but encountered a problem when a service ticket requested whitelisting of the vendor’s domains in Zscaler ZIA.  Despite no blocks in Zscaler logs, the vendor provided broad domain requests, prompting the need for a solution that adhered to security protocols

## Approach
- **Research and Analysis**:  Conducted a detailed analysis of the user's machine, the application, and Splunk logs, while reviewing Zscaler documentation to assess our security posture regarding the connection request.
- **Methodology**:  Worked closely with the requesting user and the vendor to clarify connection requirements and gather specific domain information.  Developed a systematic approach for testing the connection.  
- **Tools and Technologies**:  Utilized Zscaler Internet Access (ZIA) for traffic filtering and Zscaler Private Access (ZPA) for secure application access, analyzed connection issues through Splunk logs, and consulted Zscaler documentation for configuration guidance while collaborating with the vendor via video conferencing and email.

## Implementation
**Steps Taken**:
1. Initial Investigation:  Checked Splunk logs for the vendor's first domain and found no blocks by Zscaler; all connections were allowed.  The second domain the vendor asked us to whitelist had a wildcard: *.s3.amazonaws.com, which posed significant security risks, as whitelisting this AWS S3 domain would open our SWG up to millions of sites.
2. User Meeting:  Requested a meeting with the vendor to clarify specific domains.
3. Testing Session:  Scheduled a day for testing with the vendor and involved coworkers.  During the testing, the connection failed with Zscaler enabled but was successful when Zscaler was temporarily disabled.
4. Log Analysis:  Pulled up Splunk logs and identified that Zscaler blocked the connection due to a dropped SSL handshake.  This was because Zscaler was acting as a man-in-the-middle, interrupting the certificate pinning process enforced by the vendor's application, which led to connection failures.
5. SSL Bypass & Certificate Pinning Issue:  To resolve the dropped SSL handshake caused by Zscaler's inspection, we implemented an SSL bypass for the vendor’s specific domains.  The vendor's application used certificate pinning, ensuring it only accepted specific certificates.  Zscaler's interruption of this pinning process prevented successful SSL handshakes, which was confirmed through packet captures analyzed in Wireshark.
6.  Domain Whitelisting:  Entered the first domain provided by the vendor and re-enabled Zscaler.  The connection test was successful.
7.  Ongoing Connection Issues:  While testing addition features of the application, a connection error occurred, this time linked to one of the AWS S3 domains.
8.  Final Implementation:  Added full list of specific vendor domains to Zscaler ZIA and removed the previous SSL bypass entries, ensuring that we presented our organization's public IP instead of a Zscaler IP.  This change allowed successful connections while maintaining security.
10.  Testing Confirmation:  Conducted final tests on the application, which were successful.  The entire process, including necessary approvals and change controls, took about a month.

**Challenges Faced**:  The broad wildcard domain (*.s3.amazonaws.com) requested by the vendor posed a significant security risk, allowing unrestricted access to a wide range of AWS services.  Additionally, Zscaler's role as a man-in-the-middle disrupted the SSL handshake by interrupting the certificate pinning process, complicating the connection.  Coordinating with the vendor to provide more specific domains was critical.  Implementing an SSL bypass for specific domains was suggested by Zscaler's documentation, meaning there would be no deep packet inspection of traffic to its domains. Vendor and website validation was an imperative, along with assuring our EDR's and firewalls were in place as a contigency.   

## Results
- **Outcomes**:  Successfully implemented SSL bypass for the vendor's application, ensuring secure and seamless connections while maintaining security posture.
- **Impact**:  Enhanced the organization's security posture by eliminating the need for an SSL bypass for millions of sites.

## Lessons Learned
- **Departmental Education**:  Ensuring department heads and end-users are more aware of security protocols and potential solutions can help mitigate issues earlier in the process, streamlining troubleshooting and reducing time spent on reactive measures.
- **Proper Use of Zscaler Controls**:  Instead of temporarily disabling Zscaler, signing out of the tool during troubleshooting would reinforce proper usage and prevent end users from inadvertently learning how to bypass security controls.  This approach ensures continued adherence to security protocols without introducing unnecessary risks.

## Conclusion
- Optimizing vendor connections through Zscaler ZIA while maintaining a strong security posture required a systematic approach to troubleshoot connection issues, implement SSL bypass for specific domains, and ensure proper use of security controls. The key challenge was the vendor's request for a broad AWS S3 domain wildcard, which posed a significant risk, but with collaboration and targeted domain whitelisting, the issue was resolved. This case study highlights the importance of clear communication with vendors, the careful configuration of security tools, and the need to balance security with operational efficiency.

### References
- [ZIA SSL Inspection Leading Practices Guide](https://help.zscaler.com/zscaler-deployments-operations/zia-ssl-inspection-leading-practices-guide)
- [Certificate Pinning and SSL Inspection](https://help.zscaler.com/zia/certificate-pinning-and-ssl-inspection)
- [NIST SP 800-53 Rev. 5:  Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
    - #### Relevant NIST SP 800-53 Rev 5 Controls
      - ##### AC-17: Remote Access
          - This control highlights the importance of secure remote access to organizational systems.  By implementing source IP anchoring, you can enforce stricter access controls, ensuring that only authorized users from specific IP ranges can access vendor services.
      - ##### SC-12: Cryptographic Key Establishment and Management
          - While this control focuses on encryption, the principle of maintaining secure communications is essential.  By avoiding broad SSL bypasses, you ensure that encryption is properly enforced, thereby reducing the risk of data exposure.
      - ##### SC-28: Protection of Information at Rest and in Transmission
          - This control emphasizes the necessity of protecting data both in transit and at rest.  By utilizing source IP anchoring, you ensure that traffic to and from the vendor is inspected for malicious activity, preserving the integrity and confidentiality of sensitive information.
      - ##### SI-4: Information System Monitoring
          - Effective monitoring is vital for detecting anomalies and ensuring compliance.  By allowing Zscaler to inspect traffic instead of bypassing it entirely, you enhance your capability to monitor for malicious activity, in line with the monitoring requirements outlined in this control.
      - ##### RA-5: Vulnerability Scanning
          - This control stresses the need for regular scanning to identify vulnerabilities.  Maintaining traffic inspection through Zscaler ensures that any threats can be detected and addressed before they exploit vulnerabilities within systems.
