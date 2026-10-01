# Repair repeated review after Proceed

These seven revised topics keep the existing section cards, field bindings, topic references, and working section normalization. They replace the nested review calls with one review owner. They are local proposed replacements; they have not been installed or executed in Copilot Studio. The original snapshot in `topics/` is unchanged.

## Cause in the supplied call graph

An edit calls a collection section, which calls helper1, which calls FormValidation. Proceed returns through those callers to the editor, where `0raJQa` opens FormValidation again. Multiple edits accumulate multiple review calls. In addition, the editor only recognizes the English title Proceed.

## Changes to apply together

| Studio topic | Revised source | Change |
| --- | --- | --- |
| form1.UserRequest | [form1.UserRequest.yaml](form1.UserRequest.yaml) | Remove helper1 redirect `wj8bho`; keep EndDialog |
| form2.PreliminaryCheck | [form2.PreliminaryCheck.yaml](form2.PreliminaryCheck.yaml) | Remove helper1 redirect `m3QRXV`; keep EndDialog |
| form3.DealerInfo | [form3.DealerInfo.yaml](form3.DealerInfo.yaml) | Remove helper1 redirect `RgHPfj`; keep EndDialog |
| form4.PracticeInfo | [form4.PracticeInfo.yaml](form4.PracticeInfo.yaml) | Remove helper1 redirect `ffMKny`; keep EndDialog |
| form5.DeviceInfo | [form5.DeviceInfo.yaml](form5.DeviceInfo.yaml) | Remove helper1 redirect `Nls7Mt`; keep EndDialog |
| helper2.FormValidationEditor | [helper2.FormValidationEditor.yaml](helper2.FormValidationEditor.yaml) | Open the picker directly, normalize and route, then EndDialog. Remove the FirstTimeFormState assignment, the outer Proceed check, and recursive review redirect `0raJQa` |
| form6.FormValidation | [form6.FormValidation.yaml](form6.FormValidation.yaml) | Use stable Proceed/Edit IDs and own the review loop |

The local collection copies are based on the original supplied cards. If a live topic has additional intentional card changes, preserve those and remove only the listed helper1 redirect. Temporary TRACE messages are not needed in this repair.

## Review control

In FormValidation card `hHGJWg`, the Proceed button uses `Proceed` for its title, id, and data.actionSubmitId; the Edit button uses `Edit` for all three. The captured output remains `Global.formValidation`. The conditions use Lower and Trim and compare against `proceed` or `edit`. Either the configured ID or the observed English title now gives the same routing value. This implements the user's requested simplification. If secondary-language resources replace the titles with translated text, title-based submissions can still differ; verify the actual returned value in those languages.

The unconditional editor call `4p9dZL` becomes a conditional edit call. Proceed ends FormValidation using EndDialog. Edit calls helper2; after that topic returns, a Go to step action returns to the existing review card `hHGJWg` in the same topic. Unknown review actions produce an explanation and repeat the current card.

The user subsequently reported that the review buttons work in English, but non-English sessions return button titles in place of action IDs. A proposed custom `reviewDecision` data field was tried; the user captured `Decision=[] Submit=[Edit]` and `Decision=[] Submit=[Proceed]`. This proves the proposed decision output was not populated in the tested path, but does not establish whether the cause is localized card content, output schema, or host submission handling. That workaround has been removed. At the user's request, the revised topic now gives each button the same title and submit ID. The user confirmed that both buttons work in both tested languages and that Edit -> change Preliminary Info -> return to review -> Proceed passed. The local `debugReviewAction` message has now been removed. These are user-reported Studio results; Codex did not execute the agent.

If entering code manually, Go to step is `GotoAction` with `actionId: hHGJWg`. Use the visual editor's Topic management > Go to step if Studio flags the imported node, selecting the review card as the destination.

## Resulting path

```text
InternationalFormWorkflow
  -> request -> preliminary -> dealer -> practice -> device
  -> FormValidation
       Edit -> editor -> selected section -> return -> same review card
       Proceed -> return to InternationalFormWorkflow
  -> FormSubmission (the existing mock)
```

Do not put an End all topics action on Proceed: that would abandon the parent workflow before its FormSubmission step. An End current topic returns to the caller. [Microsoft topic management documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

The existing main workflow already calls `wf7.FormValidation` at `VFrJjd` and `wf8.FormSubmission` at `taTAMt`, so it needs no change for this repair. helper1 can remain in the agent unused by these seven topics. FirstTimeFormState is no longer used to control review in this path.

## Verification and live tests

Static checks confirm that the revised collection topics no longer call helper1, the revised editor does not call FormValidation, and the review uses two Go to step actions back to its own card. Six JSON cards were parsed: the five section cards and picker. Power Fx, full YAML schema validity, localization behavior, and actual execution remain to be checked in Studio.

After saving all seven topics, reset the test conversation to remove previous draft values and accumulated review calls. Run Topic checker, then verify:

1. Complete a fresh form and click Proceed without editing: show the mock submission once.
2. Edit Preliminary Info, submit it, then Proceed: show one refreshed review, then the mock submission.
3. Edit two different sections, then Proceed: one review after each edit, with no extra review after Proceed.
4. Cancel the picker, then Proceed: return to the review, then mock submission.
5. Repeat the cases in English and a secondary language; inspect `Global.formValidation` if routing fails. Supported values are `Proceed` and `Edit`, ignoring case and extra spaces. Record any other returned title verbatim rather than guessing translations.

This repair does not connect email or translation. Submission remains the existing mock until those actions are connected separately.
