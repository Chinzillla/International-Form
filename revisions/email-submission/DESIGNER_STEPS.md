# Build ValidateAndSend in the visual designer

Use the existing **International Form Email** flow. This guide replaces editing the condition's JSON with designer fields. The short expressions below go into individual **fx / Expression** cells; they are Power Automate expressions, not Copilot Studio Power Fx topic formulas.

The purpose is to check required data at the point where email is sent. An incomplete request receives an invalidInput response rather than reaching Outlook. Optional case/dealer/feedback fields do not block sending.

## Where to add it

1. Open **International Form Email** -> Edit.
2. Immediately after **When an agent calls the flow**, add **Data Operations -> Compose**.
3. Rename this Compose to **Clean_contact_email** before writing expressions that reference it.
4. In Compose's Inputs, open **fx / Expression**, enter `trim(coalesce(triggerBody()?['text_2'], ''))`, and insert the expression. It produces a trimmed string even if the contact email is missing.
5. Immediately after Clean_contact_email, add **Control -> Condition** and rename it **ValidateAndSend**.
6. Set the condition's group selector to **AND**. Every row must pass.

This is a Condition action inside the flow. Do not enter it as a trigger condition: a trigger that never runs cannot return an invalidInput result to the agent. [Condition designer documentation](https://learn.microsoft.com/en-us/power-automate/use-expressions-in-conditions).

## Eight required-field rows

For each row, click the left value -> **fx / Expression**, insert the expression in the table, and choose **is equal to** as the operator. For the right value, use **fx / Expression -> false** so it is a Boolean, not the word false as text.

Use **Add row** until all eight rows are present. Paste expressions without a leading @ and without JSON wrappers.

| Field | Left value: Expression | Operator | Right value: Expression |
| --- | --- | --- | --- |
| userRequest | `empty(trim(coalesce(triggerBody()?['text'], '')))` | is equal to | `false` |
| contactName | `empty(trim(coalesce(triggerBody()?['text_1'], '')))` | is equal to | `false` |
| contactEmail | `empty(trim(coalesce(triggerBody()?['text_2'], '')))` | is equal to | `false` |
| practiceName | `empty(trim(coalesce(triggerBody()?['text_9'], '')))` | is equal to | `false` |
| practicePhone | `empty(trim(coalesce(triggerBody()?['text_10'], '')))` | is equal to | `false` |
| practiceAddress | `empty(trim(coalesce(triggerBody()?['text_11'], '')))` | is equal to | `false` |
| deviceModelOrPartNumber | `empty(trim(coalesce(triggerBody()?['text_12'], '')))` | is equal to | `false` |
| deviceSerialNumber | `empty(trim(coalesce(triggerBody()?['text_13'], '')))` | is equal to | `false` |

The expression returns true for missing, empty, or whitespace-only text, so comparing it with false requires usable text. coalesce supplies an empty string for a missing value before trim runs. [Expression functions](https://learn.microsoft.com/en-us/azure/logic-apps/expression-functions-reference).

Do not add caseNumber, airTechniquesContact, dealerName, dealerBranch, dealerPhone, dealerAddress, or writtenFeedback to these required-field rows. They are optional on the form. Independently make those trigger inputs optional, as described in the phase README; this condition cannot repair a call rejected before the flow starts.

## Contact-email rows

In the same AND group, add these fourteen rows. Use fx / Expression for every left value. In the right column, Expression means use fx; Text means type the displayed character into the value field.

| Left value: Expression | Operator | Right value | Purpose |
| --- | --- | --- | --- |
| `length(split(outputs('Clean_contact_email'), '@'))` | is equal to | Expression: `2` | Exactly one @ |
| `empty(first(split(outputs('Clean_contact_email'), '@')))` | is equal to | Expression: `false` | Text before @ |
| `empty(last(split(outputs('Clean_contact_email'), '@')))` | is equal to | Expression: `false` | Text after @ |
| `last(split(outputs('Clean_contact_email'), '@'))` | contains | Text: `.` | Dot in domain |
| `startsWith(last(split(outputs('Clean_contact_email'), '@')), '.')` | is equal to | Expression: `false` | Domain does not start with dot |
| `endsWith(last(split(outputs('Clean_contact_email'), '@')), '.')` | is equal to | Expression: `false` | Domain does not end with dot |
| `outputs('Clean_contact_email')` | does not contain | Expression: `' '` | No spaces |
| `outputs('Clean_contact_email')` | does not contain | Text: `;` | No semicolon-separated recipients |
| `outputs('Clean_contact_email')` | does not contain | Text: `,` | No comma-separated recipients |
| `outputs('Clean_contact_email')` | does not contain | Text: `<` | No opening angle bracket |
| `outputs('Clean_contact_email')` | does not contain | Text: `>` | No closing angle bracket |
| `outputs('Clean_contact_email')` | does not contain | Expression: `decodeUriComponent('%09')` | No tab |
| `outputs('Clean_contact_email')` | does not contain | Expression: `decodeUriComponent('%0D')` | No carriage return |
| `outputs('Clean_contact_email')` | does not contain | Expression: `decodeUriComponent('%0A')` | No line feed |

This matches the basic email checks in the proposed JSON condition. It prevents common malformed addresses and recipient lists; it does not verify mailbox existence or fully validate every permitted email-address format.

Your condition now has **22 rows**, joined by AND: eight required fields plus fourteen email checks.

## True / Yes branch

Move the existing **Send an email (V2)** action into the True/Yes branch. Keep its connection, To/Cc, and body unless applying the separate formatting refinements in the phase README. If recreating this action on the canvas, remove its previous top-level instance so there is one email action.

For the full email proposal, add **Encode_fields** before Send an email (V2) and use its encoded outputs in the body. That formatting change is separate from entering the condition rows.

After Send an email (V2), retain/add **Respond to the agent**, renamed **Respond_Sent**. Define:

| Output name | Type | Value |
| --- | --- | --- |
| success | Boolean | true |
| status | Text | sent |

This response should run after the email action **is successful**.

For the failure-handling part of the proposal, add another response **Respond_SendFailure** in a parallel branch after the email action, not after Respond_Sent. Set its run-after dependency on Send an email (V2) to **has failed**, **has timed out**, and **is skipped**. It returns success false and status failed. The detailed layout is in the phase README.

## False / No branch

Add **Respond to the agent**, renamed **Respond_InvalidInput**, with:

| Output name | Type | Value |
| --- | --- | --- |
| success | Boolean | false |
| status | Text | invalidInput |

Leave the email action exclusively in the True/Yes branch. An invalid input then returns a result without sending email.

Every response branch must define the same output names and types. Set Asynchronous response to Off for the proposed topic that waits for send confirmation. [Agent flow response requirements](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-modify-use-with-agent).

## Save and check

Save the flow and refresh the schema at the existing email-call node in **form7.FormSubmission** after conditionGroup_weE5Pg and before Tk0LIe. Preserve that single call location.

For the full response-handling topic proposal, success maps to Topic.EmailSuccess and status maps to Topic.EmailStatus. The latest working live call was not supplied; inspect its exposed outputs rather than assuming it already uses those variables.

Run two controlled checks: complete required data reaches the email branch; an empty or whitespace-only required value returns invalidInput with the email action skipped. A required property rejected at the Call flow boundary will not reach this condition; test the condition directly in the flow designer when necessary.

This document was prepared locally. No live designer was operated or email sent by Codex.
