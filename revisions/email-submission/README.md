# Connect the completed form to email

Prepared October 1, 2026. The user confirmed that alwaysPrompt: true fixes the skipped Feedback question. This email phase is a local proposal; no email flow was executed, no email was sent, and nothing was uploaded or published by Codex.

While this proposal was being prepared, the user confirmed that the email function works after adding its call at the end of form7.FormSubmission before ending topics. Keep that working placement. The revised FormSubmission in this package uses one callEmailWorkflow node after conditionGroup_weE5Pg and before Tk0LIe; the main workflow has no additional email call. This is a proposed validation/error-handling refinement, not a requirement to replace the user's working call. If applying it, replace the existing email call rather than appending a second one. The actual new live code and flow inputs/outputs have not been supplied.

## Resulting sequence

InternationalFormWorkflow -> collection topics -> FormValidation -> FormSubmission -> optional Feedback -> EmailWorkflow -> return from FormSubmission -> CancelAllDialogs.

Proceed approves the draft but does not send immediately. FormSubmission explains that the next step sends an email containing the form details and optional feedback, with a copy to the supplied contact email. Start Feedback asks for a new written reply; the other button displays Send without feedback. Both branches return before the email helper starts.

This phase sends the original entered text. Translation is a subsequent phase; do not call the original TranslateToEnglish helper here, because it overwrites originals and translates blank feedback unconditionally.

## Apply in this order

The latest user-supplied trigger and Condition are reviewed in [Current flow review and manual corrections](CURRENT_FLOW_REVIEW.md). That supplied version has the correct required inputs and response types, but all eight right-hand condition operands still contain the field values instead of Boolean false. It also lacks a response after an Outlook send failure. Apply the two visual corrections in that review to the current live version.

### 1. Restrict International Form Email to explicit topic calls

In the agent's Tools page, select **International Form Email** -> Details -> Additional details -> clear **Allow agent to decide dynamically when to use the tool**.

Keep the tool available for explicit calls. This prevents generative orchestration from selecting it before the form runs. Other direct calls can still invoke it, so search the live agent for calls to this flow and retain the intended completed-form path.

Microsoft documents this setting as allowing only explicit topic calls when deselected. [Tool configuration](https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-tools-custom-agent).

### 2. Update the flow named International Form Email

For step-by-step condition-builder instructions, use [Build ValidateAndSend in the visual designer](DESIGNER_STEPS.md). It gives the individual fx cells and comparison settings instead of the action's JSON.

Use the existing flow with ID **88b55f37-5bbc-f111-aaaf-7ced8d42acd9**. Do not create a second email flow just for these edits.

The files in flows/InternationalFormEmail/ under this revision are **reference snippets**, not a complete solution to import. Apply the corresponding changes in the flow designer. If the designer's Code view is read-only, recreate the steps on the canvas and use the JSON to check the resulting properties.

At **When an agent calls the flow**:

- Preserve input keys text through text_14. Deleting/recreating inputs can change the keys.
- Make caseNumber, airTechniquesContact, the four dealer fields, and writtenFeedback optional.
- Keep the eight required form fields required.
- Remove the leading space in the display title writtenFeedback. Preserve the underlying key text_14.

The reference trigger is [01.WhenAgentCallsFlow.json](flows/InternationalFormEmail/01.WhenAgentCallsFlow.json). Its required array is:

```json
["text", "text_1", "text_2", "text_9", "text_10", "text_11", "text_12", "text_13"]
```

JSON Schema required controls property presence. The previous source required all fifteen properties; the earlier runtime error separately showed Studio rejected a blank required request. The revised call always maps all fifteen keys and substitutes empty strings for blank optional values.

Replace the original top-level **Send an email (V2)** and **Respond to the agent** sequence with a Condition named **ValidateAndSend**, matching [02.ValidateAndSend.json](flows/InternationalFormEmail/02.ValidateAndSend.json):

1. The Condition checks that the eight required strings contain text after trimming and that contactEmail has a basic single-address shape. It rejects common recipient separators and whitespace. This is basic format validation, not verification that the mailbox exists.
2. On the Yes branch, add a Compose named **Encode_fields**, with the object shown at actions.Encode_fields.inputs in the JSON. Each field is HTML-encoded; line endings become br tags after encoding.
3. Move the existing **Send an email (V2)** into the Yes branch after Encode_fields. Retain its existing Outlook connection and internal name **Send_an_email_(V2)**. Use the revised Subject, Body, and Cc expressions at actions.Send_an_email_(V2).inputs.parameters. To remains brandon.chin@airtechniques.com; Cc is the contact email. A blank case number produces International Support Form - New Request.
4. In Send an email (V2) -> Settings, set Retry policy to **None**, matching inputs.retryPolicy. A lost response can still make the actual send outcome uncertain; this setting avoids automatic action retries where supported.
5. Add **Respond_Sent** after the email action, configured to run after **is successful**.
6. Add **Respond_SendFailure** after the email action, configured to run after **has failed**, **has timed out**, or **is skipped**.
7. On the No branch of ValidateAndSend, add **Respond_InvalidInput**. No email action belongs in this branch.

Each Respond to the agent must define exactly the same two outputs, with the same spelling and types:

| Branch / response node | success (Boolean) | status (Text) |
| --- | --- | --- |
| Respond_Sent | true | sent |
| Respond_SendFailure | false | failed |
| Respond_InvalidInput | false | invalidInput |

Enter true/false as Boolean values, not quoted text. Set **Asynchronous response = Off** on these response actions so this topic waits for the result. Save the flow and make the updated version available to the agent before refreshing its Call flow node.

Microsoft requires consistent outputs on every response branch and documents the response time limit. This design keeps the send before the success response. [Modify a flow for an agent](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-modify-use-with-agent). Run-after statuses and retry policy are described in [Workflow error handling](https://learn.microsoft.com/en-us/azure/logic-apps/error-exception-handling).

### 3. Update the four topics

The previously working review-loop collection/editor revisions remain prerequisites. Do not restore the original section helpers that recursively open review.

| Topic | Exact location | Change and purpose |
| --- | --- | --- |
| InternationalFormWorkflow | New initializeSupportDraft before 7fYqGe | Reset collected fields and three Boolean flags for a new draft. This retains the chosen language. |
| form6.FormValidation / wf7.FormValidation | New resetReviewApproval before hHGJWg | Clear approval whenever a review topic starts. |
| form6.FormValidation / wf7.FormValidation | reviewProceedSelected, before EndDialog aDXWzi | Set Global.FormReadyToSubmit = true only on the recognized Proceed branch. |
| form7.FormSubmission / wf8.FormSubmission | 8NF3jZ and setFeedbackEndLabel | Replace mock wording with ready-to-send wording and display Send without feedback. |
| form7.FormSubmission / wf8.FormSubmission | callEmailWorkflow after conditionGroup_weE5Pg and before Tk0LIe | Use one email call after both feedback branches return. This matches the user's reported working location. |
| helper5.EmailWorkflow | Replace the topic; dpLGbg is the flow call | Check approval and required data, retain Signin, map all inputs explicitly, and handle the returned success/status. |

Full revised topic sources are in this folder. helper4.Feedback is also copied here with the confirmed alwaysPrompt repair; it does not need another live change if that revision is already installed.

Create these three **Boolean global variables** if Studio has not created them from the code: **FormReadyToSubmit**, **EmailAttempted**, and **EmailSucceeded**. The revised main topic initializes all three to false. Existing collected fields remain String globals. Save the main topic first, then the review, submission, and email topics, and run Topic checker.

**EmailWorkflow's internal schema name was not supplied.** This proposal uses brandon_Agent09232026.topic.helper5.EmailWorkflow. In FormSubmission's callEmailWorkflow node, use the canvas's **Go to another topic** picker to select the actual live **helper5.EmailWorkflow** topic; Studio will write its correct internal dialog reference. If it differs from the proposed reference, update that line and the helper5 localization prefixes in the revision copies accordingly. Do not rename unrelated topics to satisfy this assumption. If retaining the user's working direct flow call, keep its already-resolved reference and input mappings.

The revised dpLGbg uses the native InvokeFlowAction node and the known flow ID, following Microsoft's [flow-call example](https://github.com/microsoft/CopilotStudioSamples/blob/main/EmployeeSelfServiceAgent/Facilities/EmployeeCreateFacilitiesManagementTicket/topic.yaml). This avoids guessing the parameter names exposed by the unsupplied action wrapper. If Studio does not resolve the node, select the existing International Form Email flow from **Add a tool** on the topic canvas and verify the generated node and bindings against the table below.

The original Signin redirect 9ZHZ9i is retained. Its implementation was not supplied. Confirm that the configured credentials work for your intended audience; this proposal does not remove authentication or change the Outlook connection.

### 4. Verify all input and output mappings at dpLGbg

| Flow key | Display title | Explicit global source | Required |
| --- | --- | --- | --- |
| text | userRequest | Global.UserRequest | Yes |
| text_1 | contactName | Global.contactName | Yes |
| text_2 | contactEmail | Global.contactEmail | Yes |
| text_3 | caseNumber | Global.caseNumber | No |
| text_4 | airTechniquesContact | Global.airTechniquesContact | No |
| text_5 | dealerName | Global.dealerName | No |
| text_6 | dealerBranch | Global.dealerBranch | No |
| text_7 | dealerPhone | Global.dealerPhone | No |
| text_8 | dealerAddress | Global.dealerAddress | No |
| text_9 | practiceName | Global.practiceName | Yes |
| text_10 | practicePhone | Global.practicePhone | Yes |
| text_11 | practiceAddress | Global.practiceAddress | Yes |
| text_12 | deviceModelOrPartNumber | Global.deviceModelOrPartNumber | Yes |
| text_13 | deviceSerialNumber | Global.deviceSerialNumber | Yes |
| text_14 | writtenFeedback | Global.feedbackData | No |

Every input binding uses **=Coalesce(Global.variableName, "")**. Coalesce fills a blank value without replacing a nonblank user's original text. The flow trims the email address for Cc.

Flow outputs bind **success -> Topic.EmailSuccess** (Boolean) and **status -> Topic.EmailStatus** (String). Refresh the flow schema at the topic node after changing the trigger or response. Confirm all fifteen inputs and both outputs match before testing.

The emailSubmissionGate checks, in order:

1. A confirmed prior send: report already sent and return.
2. A prior attempt without confirmation: report the existing attempt and return.
3. No approved review: ask the user to complete the form and return without a send.
4. Complete required fields and a basic single contact email: continue to Signin and dpLGbg.
5. Otherwise: report incomplete details, open the working review for corrections, then recheck.

Immediately before the flow call, initializeEmailAttempt sets EmailAttempted to true. Only success = true and status = sent set EmailSucceeded to true and display emailSentMessage. Other returned results display emailUnconfirmedMessage.

A platform/action failure that prevents any response may invoke the agent's existing OnError topic instead; its source was not supplied. This local helper does not claim to catch all platform errors. Inspect that path during failure testing. Do not automatically rerun email after an uncertain response.

### 5. Upload the email-phase localization copies

Use the six JSON files in this revision's **Localizations/** folder after saving the revised topics. The existing root Localizations files remain the working demo version.

The phase copies change the two submission text blocks and the displayed End Demo label to Send without feedback. They also add translations for the new error/confirmation messages. The Start Feedback label is retained.

**English action IDs and routing values remain Start Feedback / End Demo.** The existing normalizeFeedbackAction node compares the response with the displayed label variables, so the translated Send without feedback button still routes to End Demo. Do not add one condition per language.

Download a fresh localization export after saving the topics. Check the new message paths against it, especially helper5's actual internal topic name, before uploading. These copies use the message resource pattern already present in the supplied exports; the new paths have not been verified against a Studio export.

## Tests for this phase

Use your own test contact email; the configured flow also emails brandon.chin@airtechniques.com. Codex has only prepared the local files.

| Test | Expected result |
| --- | --- |
| Fresh form -> Proceed -> Start Feedback -> distinctive written comment | Question waits; one flow invocation after the reply; email contains all collected fields and that comment; success message follows successful send. |
| Fresh form -> Proceed -> Send without feedback | No feedback question; text_14 is an empty string; send succeeds. |
| Fresh form -> Start Feedback -> type English skip | Feedback becomes empty; the request can still send. |
| Blank case number, AT contact, and all dealer fields | No required-optional-field error; default subject is New Request. |
| Spanish Start Feedback and Enviar sin comentarios | Same two English canonical routes, with translated titles and the email call in both paths. |
| Direct EmailWorkflow call without review approval | No flow invocation. |
| Approved draft with a required field deliberately blank or invalid contactEmail | Reopen review; no call until corrected and Proceed selected again. |
| HTML characters, quotes, apostrophe, and several line breaks in the request/feedback | Email displays the literal characters and intended line breaks. |
| Handled email-action failure in a controlled test | False/failed response; no success message and no automatic resend. |
| Call the email helper again for the same approved draft | Already-sent/previous-attempt branch; no second connector call. |
| New form in the same conversation | Fields and flags are reset; the new draft can be submitted. |

A connector success confirms that the send action succeeded, not that the recipient has received/read the message. The three topic flags protect this draft's helper return path; they are not durable deduplication across sessions, new drafts, connector internals, or lost responses.

Check topic references, expression types, flow schema, connection settings, run history, and localization in Studio. This package has static local checks only; it is not a verified deployment.

## Subsequent phase

Once the original-text email path passes, connect International Form Translation with explicit inputs, preserve original and translated values separately, and skip translation of empty feedback. Then finish the answer-first AI-to-form handoff and test the published channel.
