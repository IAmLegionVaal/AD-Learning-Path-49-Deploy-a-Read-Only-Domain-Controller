# AD Learning Path 49 — Deploy a Read-Only Domain Controller

## Objective
Deploy a Read-Only Domain Controller in `Branch-Site`, configure its Password Replication Policy, delegate local server administration, and verify branch authentication and read-only behavior.

## Prerequisites
- `Branch-Site` and its subnet
- Writable DC reachable during promotion
- Patched branch server with static address and internal DNS
- Separate groups for accounts allowed and denied credential caching
- Unique DSRM credential

## Setup
1. Build and join the branch server to `corp.lab`.
2. Place its network in the Branch subnet and confirm site discovery.
3. Pre-create the RODC account or run the promotion wizard directly.
4. Enable DNS and Global Catalog only when required by the design.
5. Assign a delegated local administrator for server maintenance.
6. Configure Password Replication Policy so normal branch test users may be cached while privileged identities remain denied.
7. Authenticate approved and denied test accounts and review the resulting PRP usage.
8. Verify the RODC cannot accept ordinary directory writes.

```powershell
Get-ADDomainController -Filter * |
    Select-Object HostName,Site,IsReadOnly,IsGlobalCatalog
Get-ADDomainControllerPasswordReplicationPolicy -Identity 'BRODC01' -Allowed
Get-ADDomainControllerPasswordReplicationPolicy -Identity 'BRODC01' -Denied
repadmin.exe /showrepl BRODC01
```

## Validation
```powershell
Get-ADDomainController -Identity BRODC01 -Properties *
nltest.exe /dsgetdc:corp.lab
Resolve-DnsName -Server BRODC01 -Type SRV _ldap._tcp.dc._msdcs.corp.lab
dcdiag.exe /s:BRODC01
```

## Evidence
Store RODC properties and site, PRP allowed/denied groups, sanitized caching observations, delegated local-admin design, branch-client DC locator, replication/DNS health, errors/remediation, and final pass/fail status under `evidence/`.

## Troubleshooting
- Branch user cannot authenticate during a simulated link outage: ensure the account was allowed and cached before the outage.
- Promotion fails: verify DNS, writable-DC connectivity, site mapping, and prerequisites.
- Unexpected account caching: review PRP immediately and reset affected test credentials.

## Security notes
Never allow privileged administrative identities to be cached on an RODC. Review cached-account usage and protect delegated local administration.

## Cleanup
Demote the RODC cleanly, remove stale DNS/metadata only if needed, and review any credentials that were cached during the exercise.

## References
- Microsoft Learn: Read-only domain controllers
- Microsoft Learn: RODC Password Replication Policy

## Next activity
`AD-Learning-Path-50-Build-a-Second-AD-Forest`
