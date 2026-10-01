# Bug and logic review

Reviewed October 1, 2026. Findings are based on the supplied code and linked Microsoft documentation. No live Copilot Studio session or backend flow was executed. P1 means repair before relying on the affected path; P2 means correctness or robustness work; P3 means a minor correction.

## Highest priority

### R01 — P1: Section editing compares labels instead of submitted values

**Confirmed in the supplied code.** In `topics/helper2.FormValidationEditor.yaml`, the ChoiceSet sends values such as `request` and `preliminary_info`, but the routing conditions compare `Request` and `Preliminary Info`. All five branches have this mismatch. A user can click Edit and select a section yet return to the review without changing anything.

Correct the five comparisons to the actual values:

```powerfx
Topic.SectionChoice = "request"
Topic.SectionChoice = "preliminary_info"
Topic.SectionChoice = "dealer_info"
Topic.SectionChoice = "practice_info"
Topic.SectionChoice = "device_info"
```

This is a mismatch between labels and values, not merely letter casing. Adaptive Cards defines choice `title` as display text and `value` as the raw value. [Input.Choice reference](https://learn.microsoft.com/en-us/adaptive-cards/schema-explorer/input-choice).

**Reproduce:** Complete a form, choose Edit, select each section in turn, and submit ApplyEdits. None of the section redirects should currently match.

**Subsequent user-reported runtime evidence:** English produced `Section=[preliminary_info] Action=[ApplyEdits]`, while non-English sessions produced `Section=[Preliminary Info] Action=[ApplyEdits]`. Thus the live localized path can populate the section variable with the display label rather than the raw ID expected from the supplied JSON. Comparing only the raw IDs is insufficient for both observed paths. The original reproduction expectation above applies to the raw-ID path. The point where localization or another layer changes the value has not been established.

A compatibility correction in helper2 is to insert a SetVariable after the picker and before condition group `rNT3n9`: set `Topic.SectionRoute` to `=Substitute(Lower(Trim(Topic.SectionChoice)), " ", "_")`. Compare the five conditions in `jM5hiz` against `Topic.SectionRoute` and the canonical IDs. For the supplied English labels, this converts `Preliminary Info` into `preliminary_info` and leaves existing IDs unchanged. Retain the raw variable for diagnostics. This does not map arbitrary translated labels; inspect the actual localized card definitions/resources and ensure that choice values remain stable IDs across languages.

**Confirmed cause in the subsequently supplied editor:** The user added `normalizeSectionRoute`, but all five conditions in `jM5hiz` still compared `Topic.SectionChoice`. The computed `Topic.SectionRoute` was never used by those conditions. That directly explains why `Preliminary Info` failed to enter the `preliminary_info` branch despite correct normalization. A corrected copy is saved in `revisions/helper2.FormValidationEditor.yaml`; the original snapshot remains unchanged. Live execution of this corrected copy is pending.

### R02 — P1: The answer-first handoff is absent from the supplied graph

**Confirmed snapshot gap; live-agent behavior is unknown.** ConversationStart chooses a language and sends a greeting. It neither stores the user's issue nor searches knowledge, and nothing supplied calls InternationalFormWorkflow. The workflow itself starts by asking for the issue again.

The greeting alone can be perfectly valid if agent orchestration handles the next message. The missing part is evidence of the knowledge-answer/no-answer decision and form redirect. Inspect Conversational boosting, Fallback, the orchestration mode, and any form-entry topic before adding overlapping routes. See [AI_HANDOFF.md](AI_HANDOFF.md) for the proposed connection.

### R03 — P1: Form state is neither initialized nor reset

**Confirmed initialization/reset omission.** The only assignment to `Global.FirstTimeFormState` is `false` in the editor. Each collection topic then checks that global in helper1. There is no explicit `true` at new-form entry and no clearing of previous form values.

On a second form in the same session, helper1 can reopen review after the request section while other fields still contain the previous request's data. First-run behavior also depends on the global's initialization settings, which were not supplied; do not assume an unset value safely represents the first pass. Copilot Studio can obtain uninitialized globals from their source nodes. [Global-variable lifecycle and initialization](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-variables-bot).

Initialize form state and fields at the start of each genuinely new form. Preserve the selected language and the captured AI question deliberately. Keep new-form initialization outside the edit path. `CancelAllDialogs` alone does not reset variables. [Manage topics](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

**Reproduce:** Finish one form, begin another without resetting the chat, and submit its request. Check for early review and old contact, device, optional, or translated values.

### R04 — P1: Repairing the picker exposes nested and duplicate reviews

**Confirmed call-graph problem, currently masked by R01.** After a section is edited, its helper1 opens review because the state is false. When the user chooses Proceed there, the call returns to the edited section, then to the original editor. The original editor next calls review again at `0raJQa`.

The return path is:

```text
outer review
  -> editor
     -> edited section
        -> helper1
           -> inner review -> Proceed -> return
        -> return
     -> review again at 0raJQa
```

Repeated edits deepen this chain. Proceed can require several confirmations before submission. Redirected topics normally return to their callers. [Topic redirection](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

Give one topic ownership of the review loop. Collection topics should collect data and return. Remove their calls to helper1 if the parent owns review. The editor should return the selected edit/result to that owner rather than recursively calling review. Fix this together with R01, not afterward.

The user confirmed section editing works after using the normalized variable, then reported that Proceed revisits validation. A proposed seven-topic repair is saved in `revisions/review-loop/`: remove the five collection topics' helper1 calls, let the editor return after collection, and let FormValidation loop locally on Edit and end on a stable `reviewProceed` action. Apply the revised topics together and reset the test conversation. This remains unverified in the live agent.

### R05 — P1: Button routing depends on translated display text

**Confirmed dependency; failure depends on actual localized payloads.** The review card's buttons have no explicit ID/data, and the editor only accepts the English `Proceed` string. If a localized button returns its translated title, Proceed enters the editing branch. The feedback path attempts seven display-title comparisons, which remain vulnerable to localization wording changes or a different submit identifier.

Give all routing buttons a stable ID and a matching `data.actionSubmitId` value. Localize only the title. For example:

```json
{
  "type": "Action.Submit",
  "title": "Proceed",
  "id": "reviewProceed",
  "data": { "actionSubmitId": "reviewProceed" }
}
```

Compare `Global.formValidation = "reviewProceed"`; add `reviewEdit`, `startFeedback`, and `endDemo` in the same way. Regenerate/check output schemas after changing card data. Capture real submitted output in each channel rather than assuming the fallback identifier format. Microsoft recommends distinctive submit data for consecutive cards. [Adaptive Card submit handling](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-ask-with-adaptive-card).

### R06 — P1 for production: The supplied submission is only a demo

**Confirmed, and apparently intentional.** FormSubmission displays mock completion. Neither TranslateToEnglish nor EmailWorkflow has an incoming redirect in the supplied source. Therefore this graph does not send a support request through the provided email flow.

Keep that behavior for demo testing. For production, connect validation, any optional feedback collection, translation, and email explicitly, then display success after a successful backend response. Do not infer that naming a topic FormSubmission sends anything.

### R07 — P1 when Fallback starts the form: Feedback redirects back into fallback

**Confirmed explicit redirect; downstream result depends on the missing Fallback topic.** Feedback ends by calling Fallback even though the user's feedback has already been accepted. If Fallback becomes your unanswered-question form entry, finishing feedback can start another form with the feedback or `skip` as the apparent issue.

Feedback also uses `CancelOtherTopics`. It is unsafe to rely on the normal return path to FormSubmission or the parent workflow when a child is configured to cancel other topics; confirm the stack behavior in Studio if retained.

Make feedback collect/skip and end its own topic. Let the parent control final completion. Reserve Fallback for actual unanswered support input, with a guard against reopening a form that is already active. Translate or provide a button for Skip instead of recognizing only the English word.

## Additional correctness issues

### R08 — P1 when connected: Required flow inputs have no demonstrated mappings

**Missing evidence, not proof that an action wrapper is broken.** TranslateToEnglish calls its action with `input: {}`. EmailWorkflow calls its action without an input block. The backend flows require two and fifteen strings respectively. The action definitions that could map the globals automatically were not supplied.

Inspect both action definitions and bind each input to the intended variable. If they already bind correctly, retain those mappings. If they do not, calls can fail or prompt for values unexpectedly. Use the actual exposed action input names in Studio; the backend's `text_N` keys are not guaranteed to be the topic-facing parameter names.

### R09 — P2: Optional card fields are required properties in the email contract

**Confirmed contract mismatch in intent, not automatic failure for empty strings.** Case number, AT contact, every dealer field, and feedback are optional to the user, but the email trigger requires their properties. JSON Schema `required` requires the key to exist; it does not make a string nonempty. Explicit `""` is acceptable to the shown string schema, while missing keys or null values are not.

Normalize omitted optional inputs to empty strings in the call, such as `Coalesce(Global.caseNumber, "")`, or make those properties optional in the flow contract. Apply the same handling to feedback. Decide how the subject should look when there is no case number; currently it ends with `Case `.

### R10 — P2: Translation can discard originals and couple success to optional feedback

**Confirmed overwrite and action ordering; empty-text behavior needs a live test.** The helper replaces the original request and feedback with translations. The flow always translates both, and only responds after the second succeeds. If skipped feedback is blank and its translation action rejects it, a successfully translated request still never reaches the response.

Keep original and English text in separate variables. Skip translation for blank feedback and return an empty string for it. On translation failure, preserve the original text and report/handle the failure deliberately. Select translation from actual text language or autodetection as appropriate: choosing English in the UI does not guarantee the user wrote English, and choosing German does not guarantee the request needs translation. The connector supports autodetection when source language is omitted. [Translator V2](https://learn.microsoft.com/en-us/connectors/translatorv2/).

The use of `body('Translate_UserRequest')` and `body('Translate_feedback')` is not an identified bug: this connector's Translate operation returns a string. Do not replace those expressions with an array extraction based on the separate raw Azure Translator API format.

### R11 — P2: Backend failures have no explicit failure response or retry protection

**Confirmed in the flow steps.** Both Respond actions run only after preceding actions succeed. A connector failure has no shown response branch. The email response body is empty, so the topic has no explicit status or reference to show. Retrying after an ambiguous response can send duplicate emails.

Add a handled failure path and a response contract such as `success`, `errorCode`, and `submissionId`. Show completion only after success. If retries are allowed, use a submission identifier and backend deduplication; a topic boolean alone cannot resolve an email that was sent before its response was lost. Verify response timing/settings in your actual environment rather than applying a blanket synchronous/asynchronous rule. [Current asynchronous response support](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-asynchronous-response).

### R12 — P2: Edits reopen empty inputs; selection can be empty

**Confirmed card configuration.** Collection cards do not populate Input.Text `value` with saved fields. Once editing works, a user must reenter every required field in that section. Leaving a previously populated optional field empty can clear it. The picker advertises Section(s), but defaults to one selection, and does not require one.

Use formula cards to prefill current field values. Keep one-section editing initially: change the label to Which section do you want to update?, make the choice required, and give Cancel `associatedInputs: "none"` so it bypasses validation. Reopen the picker with an explanation for a missing/unknown choice. For true multiselect, add `isMultiSelect: true` and route each returned value independently; a single ConditionGroup selects only one matching branch. [ChoiceSet defaults](https://adaptivecards.microsoft.com/?topic=Input.ChoiceSet).

### R13 — P2: Submit identifiers are mostly stored but not checked

**Confirmed omission; stale-click behavior is channel-dependent.** Several collection cards have IDs such as `continueDealer`, but their topics proceed directly to helper1 without verifying them. The first request card only uses `actionType: submitForm`, and review/feedback cards have no explicit routing data. An old card click can interfere with a later card prompt, depending on the client and payload handling.

Validate the expected action and required output fields before advancing. Distinguish different card instances during retries/edits, not just different sections, and retain a submission version if necessary. Microsoft specifically describes earlier-card clicks as a risk in consecutive-card flows. [Submit behavior](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-ask-with-adaptive-card).

### R14 — P2: Review is a preview, not complete validation

**Confirmed omission.** The preview shows empty strings for blank globals and does not validate required data again. Card requirements help the UI, but do not demonstrate protection against whitespace-only values, interruptions, stale submissions, or direct flow calls. Practice phone has no telephone input style or format validation; none of the device/name/address requirements check meaning.

Before submission, trim required strings and check for blanks, check email shape, validate the expected submit action, and validate again in the backend where appropriate. Allow international phone formatting and extensions; do not impose a US-only pattern. Decide whether serial number and practice details should be mandatory for general questions, part-number inquiries, or devices whose serial label is inaccessible. Those are product decisions rather than automatic bugs.

### R15 — P2: Selecting a locale does not provide the missing localization resources

**Configuration to verify.** The language mappings cover all seven listed choices. However, the supplied card text is English and no localization resources were included. Confirm the secondary languages are configured and that card text, titles, errors, and prompts have translations. Static Adaptive Card text is available through the localization export/import workflow. [Localize Adaptive Card content](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/localize-adaptive-cards).

Also test languages on the published channel. Language selection itself permits interruptions, so a user who asks for help before selecting a language can enter another topic and then return to the pending selector. Configure that behavior intentionally.

### R16 — P2: Raw user text is inserted into the HTML email

**Confirmed in SendEmailV2.** Fields are interpolated directly into HTML. A request containing `<b>`, `<a>`, or stray markup can change the email's presentation; literal technical text such as `x < y` can also display incorrectly. Embedded newlines are not preserved reliably as line breaks.

HTML-encode user fields before interpolation, escaping `&` first, then `<`, `>`, and quotes. Convert newlines to `<br>` only after encoding, if desired. Validate the contact email before putting it in Cc and make that copy behavior clear in the submission UI. This is an email content/recipient correctness issue; no script-execution claim is implied.

### R17 — P3: Minor source and configuration checks

- The greeting contains literal backticks, which can render it as inline code. Remove them.
- Correct `that language is not support` to `that language is not supported`.
- The email trigger title ` writtenFeedback` has a leading space. Clean it up and refresh the action schema when connecting it; do not silently rename the preserved snapshot.
- Verify the supplied Feedback snippet is actually `wf9dev.Feedback`, and that all referenced topics/actions are enabled and exist.
- If dotted labels such as `form1.UserRequest` are actual Studio display names rather than just labels in this chat, rename the display names. Microsoft warns against periods in topic names for solution export. Internal schema identifiers naturally contain periods and should not be renamed just for this reason. [Manage topics](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

## Suggested repair order

1. Fix R01 and R04 together: value-based routing and one owner of review.
2. Add explicit initialization/reset (R03) and stable routing data (R05).
3. Inspect existing boosting/Fallback settings and connect the answer-first handoff (R02).
4. Remove Feedback's Fallback redirect and unnecessary topic cancellation (R07).
5. Keep demo submission until input mappings, normalization, translation, email failure handling, and final validation are ready (R06, R08–R11, R14, R16).
6. Prefill edits, validate card actions, and check localization and channel behavior (R12, R13, R15).
