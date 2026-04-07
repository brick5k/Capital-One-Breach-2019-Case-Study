Capital One AWS Breach (2019)
Cloud Security Case Study
Executive Summary

In 2019, Capital One experienced one of the largest cloud-based data breaches in history, affecting over 100 million individuals in the United States and Canada. The attacker exploited a misconfigured AWS environment and leveraged a Server-Side Request Forgery (SSRF) vulnerability to access IAM role credentials from the AWS Instance Metadata Service. These credentials were then used to enumerate and access Amazon S3 buckets containing sensitive customer data.

This breach demonstrated several critical cloud security risks:

Misconfigured cloud infrastructure
Overly permissive IAM roles
Identity-based attacks
Insufficient monitoring and detection
Cloud data exfiltration risks

This case study analyzes how the attack occurred, what was misconfigured, why it worked, and how similar attacks can be prevented.

AWS Architecture Overview

The Capital One environment consisted of a public-facing web application hosted in AWS behind a Web Application Firewall (WAF). The application ran on an EC2 instance that had an IAM role attached, allowing access to AWS resources such as S3 buckets.

Because of improper configuration, the attacker was able to exploit this architecture.

![AWS Architecture](images/01-aws-architecture.png)

Attack Chain Overview

The attacker followed a multi-stage attack chain:

Reconnaissance
SSRF Exploitation
Metadata Service Access
IAM Credential Theft
AWS Resource Enumeration
Data Exfiltration

This type of attack is known as a cloud identity compromise, where attackers leverage legitimate credentials instead of deploying malware.

![Attack Chain](images/02-attack-chain.png)

Phase 1 — Reconnaissance

The attacker began by performing reconnaissance on Capital One’s infrastructure. This included:

Scanning endpoints
Testing HTTP requests
Identifying cloud infrastructure

During this phase, the attacker identified:

Capital One hosted infrastructure in AWS
A public-facing web application
A Web Application Firewall (WAF)
A potential SSRF vulnerability

This reconnaissance allowed the attacker to move forward with exploitation.

Phase 2 — SSRF Exploitation

The attacker exploited a Server-Side Request Forgery (SSRF) vulnerability in the web application.

SSRF allows attackers to:

Force a server to make HTTP requests
Access internal services
Bypass network controls

The attacker used SSRF to access the AWS Instance Metadata Service:

http://169.254.169.254/latest/meta-data/

![SSRF Attack](images/03-ssrf.png)

This IP address is:

Internal to AWS EC2 instances
Not accessible externally
Used to retrieve instance metadata

Because the application ran on an EC2 instance, the attacker forced the server to retrieve its own metadata.

Phase 3 — Metadata Service Exploitation

The attacker accessed:

http://169.254.169.254/latest/meta-data/iam/security-credentials/

This returned:

Access Key
Secret Access Key
Session Token

These credentials belonged to an IAM role attached to the EC2 instance.

This was the critical security misconfiguration.

Root Cause — Misconfigured IAM Role

The IAM role attached to the EC2 instance had excessive permissions.

The role allowed:

Listing S3 buckets
Reading S3 objects
Enumerating AWS resources

This violated the principle of least privilege.

Once credentials were retrieved, the attacker gained direct AWS access.

Phase 4 — Credential Abuse

Using stolen credentials, the attacker accessed AWS APIs.

The attacker was able to:

List S3 buckets
Identify sensitive data
Download files

This type of attack is known as identity-based compromise.

Because the attacker used legitimate credentials, the activity appeared normal.

Phase 5 — Data Discovery

The attacker discovered:

Customer application data
Financial records
Credit score data
Personal identifiable information (PII)

The attacker then began downloading data.

Phase 6 — Data Exfiltration

The attacker downloaded data affecting:

100 million U.S. customers
6 million Canadian customers

Exposed data included:

Names
Addresses
Credit scores
Social Security numbers (partial)
Bank account information

This resulted in a major data breach.

Why the Attack Was Successful

This attack succeeded due to:

SSRF vulnerability
Metadata service exposure
Overly permissive IAM role
Insufficient monitoring
Why Detection Failed

The attack went undetected due to:

Lack of anomaly detection
Limited IAM monitoring
No alerts for large S3 downloads
No detection of unusual API calls
MITRE ATT&CK Mapping
Phase	Technique
Initial Access	SSRF
Credential Access	Metadata Service
Privilege Escalation	IAM Role Abuse
Discovery	S3 Enumeration
Collection	Data Retrieval
Exfiltration	Cloud Storage
Security Lessons Learned
Least Privilege IAM

IAM roles should only include necessary permissions.

Avoid:

Broad S3 access
Administrative privileges
Protect Metadata Service

AWS introduced Instance Metadata Service v2 (IMDSv2) to mitigate SSRF attacks.

Monitoring and Detection

Organizations should monitor:

CloudTrail logs
GuardDuty alerts
IAM activity
Network Controls

Organizations should:

Restrict metadata access
Limit outbound traffic
Implement VPC controls
How I Would Detect This Attack

Detection methods include:

GuardDuty alerts for unusual API calls
CloudTrail monitoring
Alerts for large S3 downloads
Monitoring unusual IAM activity
Monitoring role assumption behavior
Conclusion

The Capital One breach demonstrates how cloud misconfiguration and excessive permissions can lead to large-scale data exposure. Implementing least privilege IAM, monitoring cloud activity, and restricting metadata access can significantly reduce cloud security risks.

This case study demonstrates the importance of secure cloud architecture and proper identity management in AWS environments.

Works Cited

Amazon Web Services. (2019). AWS Security Best Practices.
https://aws.amazon.com/architecture/security-identity-compliance/

Federal Bureau of Investigation. (2019). Capital One Data Breach Press Release.
https://www.justice.gov

United States Department of Justice. (2020). Capital One Data Breach Criminal Complaint.
https://www.justice.gov/usao-wdwa/press-release/file/1188626/download

Krebs, Brian. (2019). Capital One Breach Analysis.
https://krebsonsecurity.com

Amazon Web Services. (2020). Instance Metadata Service Version 2.
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html