\# \[THE PRIVILEGE AUDIT\]

\# Scenario

This investigation is an Azure IAM audit focused on reviewing who has
access to resources and whether their permissions follow the principle
of least privilege. The goal is to identify risky, excessive, or
unnecessary role assignments using different methods and document them
for the security team.

\# Environment

live multi-user Azure training tenant, Reader access.

\# Investigation

**Finding 1: IAM Blade**

1.  By downloading the CSV file with every user access listed, I needed
    to determine if anybody had more access than they should have.

<img width="975" height="304" alt="image" src="https://github.com/user-attachments/assets/42ae6e60-ad6d-4482-9a9d-7d403e085e8b" />

**Finding 2: Azure CLI**

2.  Instead of reviewing assignments manually in the portal, I used the
    Azure CLI command az role assignment list to enumerate the role
    assignments visible to my account. Reviewing the command output made
    it easier to identify unusual or orphaned role assignments that
    required further investigation

<img width="975" height="186" alt="image" src="https://github.com/user-attachments/assets/a23f826e-3d15-4572-96ba-02ac1ea1cba8" />

**Finding 3: Azure Resource Graph**

3.  I used Azure Resource Graph and KQL to search for the orphaned
    principal ID across Azure role assignments. The query allowed me to
    identify the scopes where the principle still had permissions. The
    is help me confirm whether orphaned access existed beyond the
    original resource group.

<img width="975" height="693" alt="image" src="https://github.com/user-attachments/assets/b30590ef-4879-45a6-9f3f-d36092df7817" />

**Finding 4: Privileged Identity Management (PIM)**

4.  I needed to determine which identities currently held active
    privilege access and which were only eligible to activate privilege
    roles. Using PIM, I reviewed the resource group’s roles assignments
    and exported a report showing both active and eligible assignments.

<img width="975" height="383" alt="image" src="https://github.com/user-attachments/assets/0a1422a2-8266-4964-9dd5-815ee97d735a" />

> **Finding 5: Over-provisioned Account**

5.  I identified that an over-previsioned account had been assigned the
    Owner role within a resource group that was not visible with my
    current access. I activated an eligible Azure role through PIM to
    obtain temporary authorized access, then exported the resource
    group’s role assignments and identified the account holding
    excessive
    
<img width="1036" height="217" alt="image" src="https://github.com/user-attachments/assets/c3a32633-7ebd-4223-81aa-61eb9b96769c" />

\# What broke / what surprised me

I initially expected the IAM blade to provide all of the information
needed for the audit, but I learned that different tools provide
different views of Azure access. Resource Graph was useful for searching
for role assignments across scopes while PIM was necessary for
distinguishing between active and eligible privilege access. I also had
to activate an eligible role through PIM before I could investigate one
of the resource groups.

\# Findings and recommendations

The investigation identified an orphaned role assignment and an account
with excessive Owner level permissions. These findings increase the
potential blast radius if either identity is compromised. I recommend
removing orphaned role assignments, reviewing whether Owner access if
required, and using PIM for privileged roles so elevated permissions are
granted only when needed and timed slotted.

\# What I learned

- Learned how to audit Azure RBAC assignments using IAM, Azure CLI, and
  Resource Graph.

- Learned how KQL can locate role assignments associated with a specific
  principal ID across Azure scopes.

- Learned how temporary PIM activation can provide authorized access for
  an investigation without permanently increasing privileges.
