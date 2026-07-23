STAR Answer
Situation
In DS‑Invoicing, we had a production deployment pipeline that needed access to AWS resources during deployment. At the time, I made an architecture choice to configure the pipeline using a static AWS access key and secret access key stored as a deployment secret.
The reason was speed and simplicity. It worked, it was easy to configure, and it avoided additional coordination needed for a proper OIDC-based role assumption setup.
My assumption was:

“As long as the access key is stored securely in the pipeline secret store, the risk is manageable.”

That assumption was wrong.
Later, as part of the broader AWS security compliance and secret key rotation initiative, the old access key was flagged for rotation. This aligns with the wider FinTech security work around AWS identity and secret key rotation, including DS‑Invoicing-specific key management scope. [FTG-768 AW...y Rotation | Jira Cloud], [FTG-770 AW...ng FY27 Q1 | Jira Cloud]
Someone from the team deleted the old access key and generated a new one, but the new credentials were not updated correctly in the deployment pipeline secret.
That problem only surfaced during a production deployment.
The deployment failed with an AWS access/authorization error. Because it happened during the production release window, it delayed the deployment and we missed the planned production date.
That created a lot of noise.
Security was asking why we were still dependent on long-lived access keys.
Engineering was frustrated because the deployment failed due to a credential issue, not a code issue.
Delivery stakeholders questioned why this was not identified before the production window.
And I had to accept that the root cause went back to an architecture shortcut I had made.

Task
My task was to recover trust and fix the issue properly.
I had to do three things:

restore the failed deployment path safely,
explain the real root cause transparently,
and remove the design weakness so the same issue would not happen again.

This was not just about updating a secret. The real issue was that my pipeline architecture depended on a permanent credential that could be rotated or deleted outside the deployment lifecycle.

Action
The first thing I did was own the mistake.
I did not blame the person who rotated the key. Technically, they were responding to a valid security finding. The deeper issue was that I had designed the pipeline in a way where a manually rotated static key could break production deployment.
I told the team:

“The immediate failure happened because the new access key was not updated in the pipeline secret. But the architecture mistake was mine — I should not have designed the deployment pipeline around a permanent AWS access key. We should have used OIDC-based role assumption with short-lived STS credentials from the start.”

Then I helped drive the remediation in two layers.
1. Immediate recovery
For the failed deployment, we first validated the access issue:

confirmed the old key had been deleted or disabled,
confirmed the new key existed,
checked whether the pipeline secret had been updated,
updated the pipeline secret correctly,
reran the deployment in a controlled way,
and validated that the pipeline could access the required AWS resources.

This restored the deployment path.
2. Root-cause fix
After the immediate recovery, I pushed for an architecture correction.
The proper design was:
Plain Text1Deployment pipeline2        ↓3OIDC federation4        ↓5Assume AWS IAM Role6        ↓7Temporary STS credentials8        ↓9Deploy to target AWS accountShow more lines
Instead of this weaker design:
Plain Text1Deployment pipeline2        ↓3Stored access key + secret access key4        ↓5Direct AWS accessShow more lines
The new approach removed the need to store permanent AWS credentials in the pipeline. It also reduced the risk of expired, deleted, or manually rotated keys breaking deployments.
I also proposed stronger controls:

no permanent AWS keys for deployment pipelines unless exception-approved,
owner mapping for every existing access key,
rotation checklist that includes secret-store update validation,
pre-deployment credential validation,
non-prod test before production deployment,
audit evidence for key rotation,
migration path from static keys to OIDC/STS-based access.

This aligned with the broader AWS secret key rotation/security uplift work already being tracked across FinTech and DS‑Invoicing. [FTG-768 AW...y Rotation | Jira Cloud], [FTG-770 AW...ng FY27 Q1 | Jira Cloud], [FTE-794 Se...y Rotation | Jira Cloud]

Result
The immediate deployment issue was recovered, but the bigger win was that we corrected the trust problem.
I was transparent that the failure was not just an operational miss. It was caused by an architecture decision I had made earlier.
That helped rebuild credibility because I was not hiding behind process or blaming another team.
The team moved from manually managed static credentials toward a stronger security pattern using short-lived credentials. We also introduced better key ownership and deployment readiness checks.
The lesson I took away was:

Secure storage of a permanent key is not the same as secure architecture.

For deployment pipelines, the better pattern is to avoid static credentials entirely and use OIDC-based federation with STS temporary credentials.