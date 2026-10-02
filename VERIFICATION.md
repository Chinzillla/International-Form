# Verification record

Checked October 1, 2026 against the saved files.

## Completed checks

| Check | Result |
| --- | --- |
| Topic inventory | 15 YAML topic files saved |
| Backend inventory | 7 individual JSON flow-step files saved |
| JSON flow-step parsing | All 7 parsed successfully |
| Embedded JSON Adaptive Cards | All 7 parsed successfully |
| Input IDs within each JSON card | No duplicates found |
| Basic contact-email regex samples | Accepted `support@example.com`; rejected `not-an-email` and `a b@example.com` |
| Editor submitted choice values versus routing conditions | All 5 mismatch; no matching submitted value |
| `FirstTimeFormState` initialized to true | No assignment found in supplied source |
| Incoming calls to translation/email helper topics | None found in supplied source |
| Email trigger contract | All 15 properties are required; ` writtenFeedback` leading space preserved |
| Snapshot handling | Original logic retained; recommendations written separately |

The parser checks confirm JSON syntax, not full Adaptive Cards schema conformance or rendering behavior. YAML topic text was checked structurally, including its root declaration and absence of tab indentation; a full YAML parser was not available. The Power Fx card in `form6.FormValidation.yaml` was retained as supplied and was not executed or type-checked.

## Live checks still needed

### Latest user confirmations and email proposal

The user later supplied the revised International Form Email trigger and Condition. Copies are in snapshots/2026-10-01/email-validation-as-supplied/. The trigger now requires the intended eight fields, and both response schemas have Boolean success and Text status. All eight left-side blank checks are correct, but each right side references its string input rather than Boolean false. The False branch handles invalidInput; the True branch only responds after the send succeeds and has no response configured for a send failure. Manual designer corrections are recorded in revisions/email-submission/CURRENT_FLOW_REVIEW.md. These findings are source-confirmed and have not been applied or tested live by Codex.

The user confirmed on October 1 that adding `alwaysPrompt: true` to Feedback question CF7xoW works. The exact final languages, saved written feedback value, and English skip result were not separately reported.

The user then confirmed that the email function works after placing its call at the end of form7.FormSubmission before ending topics. Record this as a successful user-reported live email path. The latest live topic source, flow run inputs, and response bindings were not supplied. Retain one email call after conditionGroup_weE5Pg and before EndDialog Tk0LIe; do not also add a call after FormSubmission in InternationalFormWorkflow.

Optional validation and error-handling revisions are saved in `revisions/email-submission/`. They use the same single-call placement, explicit mappings for all fifteen inputs, eight required form fields, approval/attempt flags, consistent flow responses, HTML encoding, and separate email-phase localization copies. The user's working live call does not need to be replaced simply because these proposals exist.

Local checks confirmed all fifteen mappings, required keys, identical response schemas, run-after failure paths, a single call in FormSubmission, no duplicate unquoted topic node IDs, and valid Goto targets. A small local expression evaluator checked twenty-four blank required-field cases, eight invalid contact-address cases, literal HTML/quotes/apostrophes, CRLF/LF/CR conversion, and missing/null/case-number subjects. All cases passed after making both subject-expression branches tolerate a missing case number. This evaluator is a local model of the used functions, not the Power Automate engine. All six phase localization copies parsed as UTF-8 JSON; comparison showed exactly three existing submission values changed and seven new message entries per file. New review instructions match the supplied translated Proceed labels. Root localization files were retained.

No full YAML parser, Studio type checker, action-wrapper schema, or fresh localization export was available for these revisions. EmailWorkflow's actual internal schema name must be selected from the live topic picker if applying the proposal. No connector execution, email send, upload, or publishing was performed by Codex. Remaining live checks include written feedback in `text_14`, empty optional fields/English skip/End Demo, handled failure, and duplicate calls for the same draft.

### Supplied localization files updated

The user supplied six localization JSON files (de-DE, es-US, ja-JP, ko-KR, zh-CN, zh-TW). An initial English-only edit was reversed when the user clarified that button titles must remain translated. Conversion of form7.FormSubmission to a Formula card preserved the card contents, output binding, and conditions, but the user still observed German titles in the action output.

The user then captured Spanish incoming payloads containing only `actionSubmitId` and `altText`, both equal to the translated title (`Finalizar demostración` or `Iniciar comentarios`). The bound output matched the payload. Thus the observed translated identifier is already present before output binding; Formula mode alone is insufficient. The responsible rendering/submission layer and the difference from the working review card remain unconfirmed.

The current proposed compatibility route uses two SetTextVariable label variables to render the translated buttons, preserves the raw action, and normalizes either an English value or the current localized label to the canonical English feedback value. The two routing conditions remain English. In all six localization files, the existing two title translations were moved to the corresponding text-variable `.Value` keys; JSON parsing and exact before/after dictionary comparison confirmed no value changes or unrelated key changes. The revised topic and its localization entries must be installed together and tested in Studio. No agent execution or full Power Fx type-checking was performed by Codex.

On October 1, 2026, the user confirmed the Spanish Start Feedback path with `Raw=[Iniciar comentarios]`, `Routed=[Start Feedback]`, `StartLabel=[Iniciar comentarios]`, and `EndLabel=[Finalizar demostración]`. The topic then displayed its Spanish thank-you message. The user subsequently confirmed `Raw=[Finalizar demostración]` and `Routed=[End Demo]` with the matching EndLabel. This confirms normalization of both Spanish buttons and the Feedback path through the thank-you node. The temporary V3 message has been removed from the saved topic. Skip behavior, other languages, and backend submission remain separate checks.

### Review repair confirmed by the user

On October 1, 2026, the user confirmed that matching the review button title, id, and data.actionSubmitId (`Proceed` / `Edit`) works in both tested languages. They then confirmed that Edit -> change Preliminary Info -> return to review -> Proceed passes. The revised sources are in `revisions/review-loop/`; the original snapshot remains unchanged. These results supersede the earlier untested status for the specific Preliminary Info edit/review path. Other section, language, backend, and published-channel checks remain separate.

The follow-on change in `revisions/feedback-completion/` uses localized label variables and normalization for the feedback buttons, returns from Feedback instead of redirecting to Fallback, and keeps skipped feedback empty. The user has confirmed both Spanish button normalizations and the Start Feedback path through the thank-you message. They subsequently reported that question CF7xoW was skipped in English and non-English sessions. Reaching the thank-you node does not establish that a written response was collected.

The saved helper4.Feedback revision now sets `alwaysPrompt: true` on CF7xoW. Microsoft documents Ask every time as the setting that asks even when the question variable already contains a value. This repair was checked for placement and preservation of the remaining topic text; it has not been executed in Studio. Test that clicking Start Feedback shows the question and waits, that a new comment is saved to Global.feedbackData, and that English skip clears the value and returns. Existing button normalization requires no changes for this repair. [Question behavior documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-ask-a-question).

### User-reported runtime observation

The user subsequently reported that the section picker returned `preliminary_info` in English and `Preliminary Info` in non-English sessions, with `ApplyEdits` unchanged in both. This is runtime evidence of inconsistent section output formats. Only the Preliminary Info example was supplied explicitly; the other sections still need equivalent multilingual checks. A proposed normalization is documented in BUG_REVIEW R01 and has not been executed in Copilot Studio.

On October 1, 2026 at 15:56:26 UTC (11:56:26 a.m. America/New_York), the user reported `FlowActionBadRequest` for International Form Email, flow ID `88b55f37-5bbc-f111-aaaf-7ced8d42acd9`. The message says the required input `text` was blank or empty at the Call flow action. The user reported that the form had not yet been invoked.

This confirms an attempted email-flow invocation with an empty request parameter. It does not identify the caller or prove that the underlying email action ran. In the supplied source, `text` is the User Request field, collected into `Global.UserRequest` by `form1.UserRequest`; the only explicit email-action call is node `dpLGbg` in `helper5.EmailWorkflow`. The live caller could be that topic, another unsupplied topic, or an agent-level tool selected by generative orchestration. Check the test activity map and the International Form Email tool's dynamic-selection setting before changing the flow's required request field.

No agent or flow was run, no email was sent, and nothing was published. The source snippets do not contain enough information to validate the complete deployed agent.

Use Studio's Topic checker to validate node fields, expressions, output schemas, and references. [Topic checker documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

Run the scenarios in [AI_HANDOFF.md](AI_HANDOFF.md), plus these targeted cases:

| Case | What to verify |
| --- | --- |
| New conversation and second form in same session | Form state initializes correctly and no old field values leak into review |
| Edit each section | The correct section opens and saves |
| Two consecutive edits, then Proceed | One final confirmation; no extra review stack |
| Edit without a section and Cancel | Clear recovery and Cancel bypasses required inputs |
| All supported languages | Prompts/cards are translated and actions retain stable identifiers |
| Feedback text, English skip, localized skip, and End Demo | Correct completion with no accidental Fallback/new form |
| Blank optional values | Email action receives strings or supported omitted properties |
| Blank feedback translation | Translation is skipped or handled successfully |
| Translation/email action calls | Inspect actual input bindings and returned outputs |
| Translation/email connector failure | A meaningful handled result reaches the user |
| Repeated/stale card clicks and retry after uncertain email response | No accidental advancement or duplicate submission |
| Request containing literal HTML characters and line breaks | Email preserves literal content and intended formatting |
| Published website/Teams channel as applicable | Behavior matches the test panel, including interruptions |

The review distinguishes source-confirmed problems from configuration-dependent risks. In particular, missing action wrappers do not prove that flow inputs are unmapped, and the absence of supplied Conversational boosting/Fallback code does not prove the live agent never answers questions.
