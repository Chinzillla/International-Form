# Review of the supplied email validation flow

Reviewed October 1, 2026. This records the trigger and Condition supplied in chat, rather than applying changes to the live flow. The flow designer's code view is read-only for the user; all corrections below use the visual designer.

## Correct parts

- When an agent calls the flow keeps eight required fields: User Request, contact name/email, practice name/phone/address, and device model/serial.
- Case number, AT contact, four dealer fields, and feedback are optional in the supplied trigger.
- All eight left-hand blank-check expressions correctly combine empty, trim, and coalesce.
- Both response schemas declare success as Boolean and status as Text.
- Respond_to_the_agent_Success returns true/sent after Send_an_email_(V2) succeeds.
- Respond_to_the_agent_Failure is in the Condition's False branch and returns false/invalidInput.
- The supplied Condition contains one email action, confined to its True branch.

## Required correction 1: right-hand values on all eight condition rows

The supplied condition compares each Boolean blank-check result with the actual text of that field. For example, the first comparison uses the blank check on the left and the userRequest value on the right. The right-hand side must be the Boolean false.

In **International Form Email -> Condition / ValidateAndSend -> Parameters**:

1. Keep the AND group.
2. Keep all eight left-side expressions.
3. Keep **is equal to** on every row.
4. In each row's **right-hand box**, click X on the existing dynamic-content token.
5. Click that same right-hand box -> **fx / Expression**.
6. Enter **false**, without quotes and without a leading @.
7. Select Add/Insert/Update, according to the designer's button label.
8. Repeat for userRequest, contactName, contactEmail, practiceName, practicePhone, practiceAddress, deviceModelOrPartNumber, and deviceSerialNumber.

The right side of each row should now be a purple fx false token. It should no longer contain the field token.

For nonblank text, the blank check returns false and comparison with false passes. For missing, empty, or whitespace-only text, it returns true and comparison with false fails. The AND group requires every required field to pass.

Do not change the existing False-branch response's false value: that output is already correct.

[Condition expressions](https://learn.microsoft.com/en-us/power-automate/use-expressions-in-conditions).

## Required correction 2: handle a send failure inside the True branch

The existing **Respond_to_the_agent_Failure** handles invalid form data. It does not run when Outlook fails: the Condition has already chosen the True branch in that case, and Respond_to_the_agent_Success is skipped.

Inside **ValidateAndSend -> True/Yes**:

1. At the plus sign immediately after **Send an email (V2)**, add a parallel branch.
2. Add **Respond to the agent** in that branch. Rename it **Respond_to_the_agent_EmailFailure**.
3. Add **success** as **Boolean / Yes-No**, with value entered through fx as **false**.
4. Add **status** as **Text**, with the plain value **failed**.
5. Open the new response's **Settings -> Run after** (or its three-dot menu -> Configure run after in the classic designer).
6. Set its only predecessor to **Send an email (V2)**.
7. Select **Has failed**, **Has timed out**, and **Is skipped**, then clear **Is successful**.
8. Leave Respond_to_the_agent_Success dependent on Send an email (V2) **Is successful** only.

The three response outcomes then are:

| Outcome | Response | success (Boolean) | status (Text) |
| --- | --- | --- | --- |
| Required data fails validation | Respond_to_the_agent_Failure | false | invalidInput |
| Outlook send succeeds | Respond_to_the_agent_Success | true | sent |
| Outlook send fails/times out/is skipped within the True branch | Respond_to_the_agent_EmailFailure | false | failed |

All three responses must use the same output names and types. For the proposed topic that waits for the send result, set Asynchronous response to Off.

[Run-after error handling](https://learn.microsoft.com/en-us/azure/logic-apps/error-exception-handling), [Agent flow response contract](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-modify-use-with-agent).

## Follow-on refinements

These are separate from the two immediate corrections:

- The current condition checks presence only. The email-format checks in DESIGNER_STEPS.md have not been added in this supplied version.
- The HTML body interpolates raw user text. The proposed Encode_fields action in the email phase preserves literal HTML characters and line breaks; it is not present in this supplied version.
- A blank caseNumber currently leaves the subject ending in Case. A conditional subject can give it a New Request fallback.
- The existing writtenFeedback input title still begins with a space. In the trigger, edit that existing label to writtenFeedback; keep its underlying text_14 key.
- No retry override is shown for Send an email (V2). To avoid automatic action retries after an uncertain send result, set its Settings -> Retry policy to None where supported. This does not guarantee durable deduplication or recipient delivery.

## Save and verify

Save the flow, then refresh the existing email call in **form7.FormSubmission**, after conditionGroup_weE5Pg and before Tk0LIe, to load its response schema. Keep one call in that location.

A complete-input test should enter the True branch and return true/sent after a successful email action. A whitespace-only required-field test should enter False and return false/invalidInput when it reaches the flow; an input rejected by the call/trigger cannot exercise this branch.

No email was sent or live configuration changed by Codex. Local verification covers JSON structure, required keys, mismatched right operands, response schemas, and missing error-response coverage. It does not execute Power Automate's expressions or validate the live connector.
