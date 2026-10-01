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

### Review repair confirmed by the user

On October 1, 2026, the user confirmed that matching the review button title, id, and data.actionSubmitId (`Proceed` / `Edit`) works in both tested languages. They then confirmed that Edit -> change Preliminary Info -> return to review -> Proceed passes. The revised sources are in `revisions/review-loop/`; the original snapshot remains unchanged. These results supersede the earlier untested status for the specific Preliminary Info edit/review path. Other section, language, backend, and published-channel checks remain separate.

The next proposed change is in `revisions/feedback-completion/`: match the mock-submission button titles and IDs, return from Feedback instead of redirecting to Fallback, and keep skipped feedback empty. It has not been installed or tested in Studio.

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
