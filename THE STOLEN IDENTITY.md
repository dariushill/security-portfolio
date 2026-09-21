THE STOLEN IDENTITY

# Scenario

Sometime in the last 24 hours, someone got access into the Mad Hat Labs
tenant. They didn't kick down any doors using any zero-day exploits on
anything. They walked right in through the identity plane and were quiet
about it. No alarms were tripped. The Mad Hat Labs security team were
NOT notified of any MALFEASANCE. The logs just show a series of
perfectly normal sign-ins.

# Environment

live multi-user Azure training tenant, Reader access.

# Investigation

Entry:

So far, we have determined that an employee was a victim of an OAuth
phishing attack. The attacker targeted the employee with a portal that
looked almost identical to the portal that the employee normally login
into. The employee information at that point was stolen and the attacker
used that information to breach the company data.

1.  To start my investigation, I signed into azure portal and navigated
    app registrations to see if any notes were input inside the note’s
    property. The reason for checking the note property is because every
    app registration carries a internal notes property under branding
    and property inside Azure portal.

<img width="975" height="610" alt="image" src="https://github.com/user-attachments/assets/72d6df61-ee37-4b67-93f5-a2f25dfe70ca" />

Escalate:

Since Carl company credentials were compromised, I need to look at what
privileges Carl currently have. I know Carl is an Accountant for the
company, so he didn’t have privilege on their account, but the legacy
app he did. The bad actor can escalate their privileges immediately by
using Carl’s owner rights to drop a new credential into the app. The bad
actor can create a new password for Carl and elevate their access from
standard to highly privileged granting access to the app directory

2.  I access Certificate and Secrets on the legacy app and notice that
    the secret that the secret created has expired 12/31/209. It shows
    that the bad actor is preparing to stay for a long time.

<img width="975" height="434" alt="image" src="https://github.com/user-attachments/assets/5ebe0537-b63b-4e88-b63b-326848e5c355" />

Pivot:

The bad actor knows that the stolen secret from Carl can be easily
rotated to a new secret which would stop the attacker in his/her tracks.
For the bad actor to have a more perinate resident inside
Mad-Hat-Legacy-Sync-Service app, the bad actor created a new app
(Mad-Hat-Labs-App). Whenever creating an App Registration in Entra ID,
Microsoft also creates a Service Principle for that application. The bad
actor took the service principle belonging to Mad-Hat-Labs-App and made
it the Owner of the Mad-Hat-Legacy-Sync-Service app. Looking at the API
permissions for the Mad-Hat-Legacy-Sync-Service app, the permissions
were granted with admin consent. This also give me an idea of the
potential blast radius of the attack.
<img width="975" height="447" alt="image" src="https://github.com/user-attachments/assets/396607c5-54b6-4403-ac01-0a1b7078d653" />

3.  I investigated the suspicious app for any notes inside the notes
    field in branding and properties blade.

<img width="975" height="602" alt="image" src="https://github.com/user-attachments/assets/16da1e50-7130-4b62-b80b-51a124d72def" />

Persistence:

The bad actor is probably aware that standard practice for accounts or
services that was implicated in a breach of secrets will be rotated.
This bad actor had a back up plan to maintain access if the secret
rotation occurred. The bad actor used Expose an API on the
Mad-Hat-Legacy-Sync-Service app. Expose an API tells Entra ID that other
apps are allowed to request permission to call my app.

4.  I went inside the legacy app and Open Expose an API blade and see
    the permission that the bad actor created.

<img width="975" height="546" alt="image" src="https://github.com/user-attachments/assets/0e551318-f132-4cd9-931c-1a5506bf14e8" />

The Loot:

Now that the bad actor is in and a back plan was created to keep access
if secrets rotate. The bad actor focused on token that proves Microsoft
already trusts him/her. A redirect URI is the address Microsoft sends
you back to after logging in. The bad actor now controls the redirect
address.

5.  

<img width="975" height="516" alt="image" src="https://github.com/user-attachments/assets/754cb064-8bb8-4d24-a5aa-fe7ee143eb4e" />

\## What broke / what surprised me

I didn’t realize how powerful OAuth can be until working through this
lab. What surprised me was how the bad actor chain of actions could
start with OAuth application permissions rather than simply stealing a
password.

\## Findings and recommendations

Findings:

One finding was that a legacy application had highly privileged API
permissions, including Directory.Read.All and User.Read.All. These
permissions created a significant risk because the application had broad
access to directory information and user information. Another finding
was the use of a long-lived client secret. The secret had an expiration
date set far into the future, which could allow an attacker to maintain
access for an extended period of if the credential was not discovered
and removed. The investigation also identified an additional application
owner. This created a persistence risk because the owner may be able to
manage the application and potentially create new credentials even after
an existing malicious secret was removed. A suspicious redirect URI was
also identified on the rogue application. The redirect URI pointed to
attacker-controlled infrastructure, meaning authorization codes
generated during the OAuth flow could be delivered to the
attacker-controlled application.

Recommendations:

1.  Organizations should regularly review app registrations and
    enterprise applications for unnecessary or excessive API
    permissions. Highly privileged permissions such as
    Directory.Read.All and User.Read.All should only be granted when
    there is a documented business requirement.

2.  Client secrets and certificates should be reviewed frequently.
    Long-lived credentials should be avoided, and the organization
    should use shorter expiration periods and credential rotation
    whenever possible

3.  Application ownership should also be audited. The organization
    should verify that every owner is still required and that former
    employees, unused service principles, and unknown applications are
    removed from ownership roles.

# What I learned

1.  How attackers can abuse OAuth. An attacker doesn’t always need to
    steal a user password or log in directly as that user. If a user
    consents to a malicious application, the attacker may be able to
    obtain delegated access through OAuth with resources on the user’s
    behalf.

2.  Highly privileged API permissions can create serious risk,
    especially when they are assigned to older or poorly monitored
    applications. Permissions such as Directory.Read.All and
    User.Read.All can give an application broad access across the
    tenant.

3.  Long-lived client secrets and additional application owners can
    allow an attacker to maintain control even after the attacker
    created secret is removed. This showed that sometimes simply
    changing a password or removing a secret may not fully remove an
    attacker from the environment.

4.  I would investigate the full relationship between users,
    applications, service principles, permissions, and resources.
