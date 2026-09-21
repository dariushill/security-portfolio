 \[Title: OPERATION DEAD DEPLOY\]

# Scenario

A junior intern at Mad Hat Labs created a test environment for an
experiment that wouldn't be presented to leadership. Intern wasn't
familiar with the company's governance standards and cut several
corners. The intern deployed their environment and left the office for
the weekend. I came in on Monday morning and found the environment the
intern created. I need to access the damage, identify what governance
failed, and document the evidence.

# Environment

Live multi-user Azure training tenant, Reader access.

# Investigation

- I logged in and looked at the resource groups to see if anything stood
  out. Resource group (testdeploy123) stood out compared to the other
  resource groups naming conventions.
  
<img width="1013" height="69" alt="image" src="https://github.com/user-attachments/assets/2ff41791-5e22-4091-8106-a671ae401d98" />

- Inside resource group testdeploy123 only one resource created was a
  storage account.
  
<img width="1013" height="125" alt="image" src="https://github.com/user-attachments/assets/f1dc628e-e9ac-4aa7-b717-ef3fa54fdcc9" />


- Inside the resource, the owner of the resource created tags.

<img width="874" height="83" alt="image" src="https://github.com/user-attachments/assets/21ab1a1c-4068-424c-9a26-a70c755b26b3" />


- Next, I needed to figure out the deployment that created the resource.

<img width="1013" height="94" alt="image" src="https://github.com/user-attachments/assets/d6c4dbca-1219-49f4-b813-ad7d42bfd2e0" />

- I needed to figure out how the resource group and resource were
  created when there is a policy in place for resource group naming
  convention. After further investigation I found that the policy in
  question effect type was set to audit. This allows the resource group
  to be created even if the naming convention wasn’t followed. Instead
  of not allowing the user to create the resource after violating the
  policy, it flagged the resource group as non-compliant.

<img width="1013" height="260" alt="image" src="https://github.com/user-attachments/assets/c053f207-f463-4206-897b-0e6dcc6d91ad" />

# What broke / what surprised me

I encounter issues at several points in these investigations. First
issue I encountered was the resource group naming convention. When I did
a quick scan of the available resource group, two stood out to me as a
potential suspect to be further investigated. One of the resources in
question didn’t have any resources created. The other resource group in
question did. I still had doubts, so I looked at Microsoft docs for
Azure resource group naming best practices and quickly noticed that one
in question was the correct resource group that needed further
investigation.

The second issue I encounter is the policy. I saw that there were
policies created and one policy for naming convention. I couldn’t figure
out how the resource group was created instead of being denied by the
naming convention policy. I saw that the resource group was flagged but
was still allowed to be created. I noticed that the naming convention
policy effect type was set to audit. I looked through the Azure
documents and discovered that audit doesn’t stop the resource group or
resource from being created but instead flagged it as non-conformant.
This is the reason how the user was able to create the resource group.

# Findings and recommendations

- Update naming convention policy effect type from “audit” to “deny”

- This would be a learning experience for the intern who created the
  resource group. Educate the intern on what he/she did wrong.

- Speak with the administrator who gave the intern temporary contributor
  access.

# What I learned

- In this investigation I learn about resource hierarchy (management
  group, subscription, resource group, and resources) and naming
  discipling
  (/subscriptions/{subscription-id}/resourceGroups/{rg-name}/providers/{provider}/{resource-type}/{resource-name})

- I learn about the importance of tags. Tags can provide valuable
  information about resources in question.

- I learn about the importance of deployment records. The deployment
  record can provide valuable information like who created the resource.
  You can correlate the deployment timestamp with security alerts.

- I learn about Azure Policy, audit and deny. These two rules’ effects
  used to control how Azure responds when a resource doesn’t follow
  organizational rules. Audits allow the resource to be created or
  changed, but Azure flags it as “non-compliant” so administrators can
  see and investigate the violation. I think of it as “I’ll allow it,
  but I’m going to report it”. Deny, on the other hand, prevents the
  resource from being created or changed if it violates the policy. I
  think of it as “this doesn’t follow our rules, so I’m stopping it”
