# THE STOLEN IDENTITY
## Investigating a threat actor gaining privileged access.

## Scenario
A user was phished and entered their credentials into a fake login page, then completed MFA. That handed an authenticated session token with an MFA-satisfied claim straight to the threat actor, letting them sign in normally and bypass Conditional Access Policies without triggering alerts. Because outdated access controls left this standard user as an Owner of a legacy enterprise app, the intruder had a direct path to escalate privileges. My job was to reconstruct the intrusion across the app registration blades using read-only access.

## Environment
Live multi-user Azure training tenant, Reader access.

## Investigation
Stage 1 (Entry): First, I needed to verify how the intruder got their foot in the door when sign-in logs looked completely legitimate. Every app registration has a free-text "Internal notes" property under Branding & properties, which the incident team used to log initial triage context. Reading that note confirmed the foothold came from a phished accounting user whose MFA-satisfied session token bypassed Conditional Access, combined with leftover Owner permissions on the legacy app. 

<img width="1624" height="849" alt="Stolen_Identity_1" src="https://github.com/user-attachments/assets/a23674fc-0e8d-488b-a7dc-2320654cde3f" />


Stage 2 (Escalate): Next, I checked the API permissions and Certificates & secrets blades on Legacy-Sync-Service. While the compromised user did not hold privileged directory roles themselves, the legacy app carried admin-consented Microsoft Graph application permissions (Directory.Read.All and User.ReadWrite.All). Using the user's Owner rights, the attacker generated a new client secret set to expire nearly a century out, letting them use the Client Credentials flow to authenticate directly as a privileged Service Principal without ever needing the human account again. 

<img width="1712" height="857" alt="Stolen_Identity_2" src="https://github.com/user-attachments/assets/ed60b2c5-5a1d-430b-939d-cb58184b0e1f" />


Stage 3 (Pivot): A single client secret is easy to cut off if defenders rotate credentials during a response, so the attacker built a backup path. Relying on Entra ID's default setting that allows standard users to register applications, they created a rogue app (Mad-Hat-Labs-App) and added its Service Principal to the Owners list of Mad-Hat-Legacy-Sync-Service. This turned the rogue app into a shadow admin that can generate fresh credentials on the legacy app at any time, even if the original user account is locked down. 

<img width="1875" height="530" alt="Stolen_Identity_3" src="https://github.com/user-attachments/assets/4a9ecf5c-46d9-451c-a306-a6b51d8d5251" /> 

<img width="1629" height="377" alt="Stolen_Identity_3 2" src="https://github.com/user-attachments/assets/33573daa-ab55-4ac8-8648-7a889ed74c9b" /> 

<img width="1718" height="562" alt="Stolen_Identity_3 3" src="https://github.com/user-attachments/assets/0c686782-a5b2-4d58-bfc1-1ada7ef04251" />

Stage 4 (Persistence): To survive even if their ownership or secrets were removed, the attacker configured the Expose an API blade on the legacy app and injected a custom scope. Exposing an API tells Entra ID that the legacy app can act as a protected backend resource for other apps to call. This created a trust bridge that their rogue app could abuse in a separate follow-up campaign. 

<img width="1641" height="864" alt="Stolen_Identity_4" src="https://github.com/user-attachments/assets/2e5e93a5-1a62-4b7f-8e2c-0d9ee3f0475f" />

Stage 5 (The Loot): Finally, I checked the Authentication blade on Mad-Hat-Labs-App and spotted a suspicious external redirect URI alongside a standard local-development URL. By combining the rogue app's client ID, the custom API scope, and that redirect URI, the attacker could craft an OAuth phishing link to harvest delegated tokens from users already signed in on corporate devices. Once a victim accepts the prompt, Entra ID creates a hardcoded OAuth2PermissionGrant, allowing the rogue app to trigger the legacy app's privileged backend ("confused deputy") while surviving password resets, session revocations, and MFA enforcement. 

<img width="1881" height="857" alt="Stolen_Identity_5" src="https://github.com/user-attachments/assets/c01c57c6-2e7d-4fbe-8d05-3b20696cd89c" />


## What broke / what surprised me
What surprised me most was how quietly the entire kill chain played out without any zero-day exploits or triggered alerts, blending into normal sign-ins and Entra ID configuration changes. It was also eye-opening to see how persistent OAuth consent abuse is: standard incident response steps like resetting passwords, revoking sessions, or enforcing MFA will not clear a malicious OAuth2PermissionGrant once a user consents.

## Findings and recommendations
- Enforce least privilege on app ownership and permissions: Remove unnecessary human owners from legacy app registrations and regularly audit Microsoft Graph application permissions like User.ReadWrite.All to limit the blast radius.

- Lock down app registrations and user consent: Turn off the default Entra ID setting that allows standard users to register new apps, and require admin approval before users can grant OAuth consent to unverified applications.

- Audit credential lifecycles and app properties: Enforce short expiration limits on client secrets (or move to managed identities/certificates) and routinely monitor app registrations for unfamiliar Service Principal owners, newly exposed APIs, and external redirect URIs.

## What I learned
Identity properties on app registrations act just like firewall rules, except teams often treat them like background documentation rather than an active security perimeter. Reviewing direct roles like Global Admin is not enough on its own. You have to audit the full chain of app ownership, API permissions, and OAuth grants to catch shadow admins before an attacker exploits them.
