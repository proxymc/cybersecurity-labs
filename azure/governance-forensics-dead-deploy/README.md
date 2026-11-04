# OPERATION DEAD DEPLOY
## Investigating a non-compliant deployment

## Scenario
An intern with temporary Contributor access deployed a "test environment" over a weekend, cut every corner, and left. I arrived on Monday and had to investigate why our governance policy did not stop it.

## Environment
Live multi-user Azure training tenant, Reader access.

## Investigation
Stage 1: I opened the Azure Home dashboard and from there I went to Resource groups. I immediately discovered that one of the groups is misnamed and doesn't follow Microsoft's established naming convention by lacking the "rg-" prefix, environment, and region indicators.

Stage 2: I investigated further by opening the offending resource group. The first action to uncover more details about this deployment is checking the Tags blade. I discovered a new tag added by the intern with typed instructions used when documenting a test resource. 

Stage 3: I confirmed this resource was created via an Azure Resource Manager deployment. When inspecting the Deployments blade, I found one payload that also did not follow the deployment’s naming convention. The exact timestamp and name gave strong evidence that the payload was deployed by the intern.

Stage 4: Knowing that the resource group naming convention policy is supposed to enforce the rule, I further investigated the "Policies" blade. The Naming Convention rule in the Compliance blade shows as Non-Compliant, where further investigation of the Assignment blade shows that the policy has a value set to "Audit". This policy only flags non-compliant requests but it allowed our intern to deploy the offending resource. 

## What broke / what surprised me
This was a fairly straightforward and simple investigation. However, prior knowledge of the tenant’s policy effects helped to pinpoint the exact reason why the misnamed resource group was allowed to exist.

## Findings and recommendations
The naming convention policy was active but configured with an "Audit" effect. I would suggest immediately updating the Naming Convention policy assignment effect from "Audit" to "Deny". I would also review the RBAC policy. Giving an intern privileged access to deploy new resources over the weekend is risky and exponentially widens the blast radius. Even if a policy remains in "Audit" mode, the tenant owner should receive an immediate email or webhook notification when a non-compliant resource is deployed, rather than waiting for someone to manually check the Compliance blade on Monday.

## What I learned
- The distinct difference between tracking compliance ("Audit") and enforcing governance ("Deny").
- Tags are not just for billing; they act as breadcrumbs left by the creator. Discovering the typed instructions in the tag helped confirm the resource's purpose (a test) and the user's intent.
- I also learned the differences between deployment and resource group naming conventions.
- Clicking through individual resources and blades is not efficient, especially with an enormous database or environment. It would be easier to inspect Activity Logs with chronological timestamps or refer to a visual representation in Azure Resource Graph.
