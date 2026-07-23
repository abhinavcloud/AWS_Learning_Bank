Yes — this is the nuance. The remediation was **not simply “remove all alphanumeric characters from payload manually.”** It was more careful than that.

## What remediation actually happened

From [R2607.R4 Deployments](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d\&EntityRepresentationId=e34e3ea7-0d1a-40d0-8ffe-747f8273fc58), the agreed immediate remediation was:

* **Roll back / remove the recent validation logic** for bank-account `last4` because it was causing production failures. [\[R2607.R4 Deployments \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d)
* **Manually remediate incidents caused by spaces or special characters** in `last4` by removing the space and replaying the affected events. [\[R2607.R4 Deployments \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d)
* **Create a DB script** to update failed events caused by the `last4` validation so they could be remediated and replayed. [\[R2607.R4 Deployments \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d)
* **Check with Payments SME/PO** whether non-digit characters such as `943s` were expected, because the data contract was not fully clear at that point. [\[R2607.R4 Deployments \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d)

So the safe wording is:

> “For known failed space/tab cases, the workaround was to remove the whitespace from the failed event and replay it. For alphabetic cases like `943S`, the team did not blindly strip the alphabetic character; those incidents were routed to Payments to confirm whether that value was valid or whether Payments needed to fix the upstream data.”

That is supported by [mpan/ last4 issue in RCP flow](https://teams.microsoft.com/l/message/19:meeting_NDA5MDdiMTItYTNjNy00NjllLWI2ZGUtNmQ1NTUzODNiZDU5@thread.v2/1784628989915?context=%7B%22contextType%22:%22chat%22%7D\&EntityRepresentationId=297b229b-1d9d-4567-af2b-d4e4a60e62ba), where [Prakash Kumar](https://www.office.com/search?q=Prakash+Kumar\&EntityRepresentationId=bfb9a4d9-37ee-412d-a74a-7db691ba5b39) assigned two incidents to Payments: one for alphabet character `"943S"` and one for a leading space `" 665"`, and explicitly asked whether these were expected and whether bank-account `last4` should be numeric or alphanumeric. [\[mpan/ last...n RCP flow \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NDA5MDdiMTItYTNjNy00NjllLWI2ZGUtNmQ1NTUzODNiZDU5@thread.v2/1784628989915?context=%7B%22contextType%22:%22chat%22%7D)

## Did we remove alphanumeric characters manually?

**I would not say that.**

What I can confirm:

* For a space case, the meeting transcript says the value looked like `665`, but it actually had a leading space; the guidance was that the space could be removed and the event retried. [\[R2607.R4 D...Recording \| Video\]](https://teamtelstra-my.sharepoint.com/personal/prakash_kumar_3_team_telstra_com/_layouts/15/viewer.aspx?sourcedoc={c2d161a3-13e9-4a08-8e6f-59e337c1b724})
* For alphabetic value `"943S"`, the source says the incident was assigned to Payments to confirm whether it was expected or needed a fix. It does **not** say DS‑Invoicing manually removed the `S`. [\[mpan/ last...n RCP flow \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NDA5MDdiMTItYTNjNy00NjllLWI2ZGUtNmQ1NTUzODNiZDU5@thread.v2/1784628989915?context=%7B%22contextType%22:%22chat%22%7D)
* Payments later responded that, as per the BECS file, **alphanumeric and spaces are allowed values for bank account number**, and that BT was sending `\t` replacing the space character in events. [\[mpan/ last...n RCP flow \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NDA5MDdiMTItYTNjNy00NjllLWI2ZGUtNmQ1NTUzODNiZDU5@thread.v2/1784628989915?context=%7B%22contextType%22:%22chat%22%7D)

So defensible answer:

> “We manually cleaned whitespace cases for immediate replay, but did not assume all alphanumeric values were invalid. Alphabetic cases were escalated to Payments/data-contract clarification.”

## How did we decide the validation was not correct?

The decision came from production reality + unclear contract.

Initially, the validation was added because the team believed bank-account `last4` should contain only digits, and FTECH-59189 Last4 validation added inside rm says Payments was expected to fix values like `909\t` to `909`; DS‑Invoicing added validation so invalid values failed early and did not generate JSON, because later remediation would become complicated. [\[FTECH-5918...inside rm \| Jira Cloud\]](https://telstra.atlassian.net/browse/FTECH-59189)

But during [R2607.R4 Deployments](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d\&EntityRepresentationId=e34e3ea7-0d1a-40d0-8ffe-747f8273fc58), the team found that real production data had spaces/tabs/special characters, and validating strictly was causing receipt failures. The team specifically said they did not know how many accounts had spaces, and decided to roll back/reduce the validation to avoid wider production incidents. [\[R2607.R4 D...Recording \| Video\]](https://teamtelstra-my.sharepoint.com/personal/prakash_kumar_3_team_telstra_com/_layouts/15/viewer.aspx?sourcedoc={c2d161a3-13e9-4a08-8e6f-59e337c1b724}), [\[R2607.R4 Deployments \| Meeting\]](https://teams.microsoft.com/l/meeting/details?eventId=AAMkAGJmNTdmYmQ2LTViOWUtNDBkYi04ZjUzLTljMmZhZDVlNDk5YQBGAAAAAAAp0w8fzw8aTJ9cujw3uxraBwA11CFl38WiSIs55UGBGDZrAAAAAAENAAA11CFl38WiSIs55UGBGDZrAAblY7hLAAA%3d)

Then [mpan/ last4 issue in RCP flow](https://teams.microsoft.com/l/message/19:meeting_NDA5MDdiMTItYTNjNy00NjllLWI2ZGUtNmQ1NTUzODNiZDU5@thread.v2/1784628989915?context=%7B%22contextType%22:%22chat%22%7D\&EntityRepresentationId=297b229b-1d9d-4567-af2b-d4e4a60e62ba) added the key clarification from Payments: according to the BECS file, **alphanumeric and spaces are allowed for bank account number**. That meant the assumption “bank-account last4 must be numeric only” was too strict for real production data. [\[mpan/ last...n RCP flow \| Teams\]](https://teams.microsoft.com/l/message/19:meeting_NDA5MDdiMTItYTNjNy00NjllLWI2ZGUtNmQ1NTUzODNiZDU5@thread.v2/1784628989915?context=%7B%22contextType%22:%22chat%22%7D)

## Simple interview explanation

Use this:

> “The initial fix assumed bank-account `last4` should be numeric only, so validation was added to fail events early when spaces, tabs, or special characters appeared. That looked correct based on our original understanding, but production data showed real bank-account values could contain spaces or alphanumeric characters. Because we did not know the full customer impact, we chose a fast workaround: remove/rollback the strict validation, clean known whitespace failures, and replay affected events. For alphabetic cases like `943S`, we routed the issue to Payments instead of blindly stripping the character, because we needed confirmation of the actual data contract.”

## Strong bias-for-action line

> “I did not wait for the final data-contract RCA because the production failure mode was already clear: our validation was blocking legitimate-looking production events. The reversible action was to relax the validation, remediate known failed events, and replay them, while Payments confirmed the correct contract in parallel.”
