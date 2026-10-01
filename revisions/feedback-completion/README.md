# Return normally after mock submission feedback

Apply these two proposed replacements after the working review-loop changes. Originals in `topics/` remain unchanged. No email or translation action is added in this step.

## form7.FormSubmission

At the beginning of this topic, `initializeFeedbackData` clears feedback for this form. Card `8NF3jZ` now uses a Power Fx formula, matching the working FormValidation card's format. Its primary-language title, id, and data.actionSubmitId are `Start Feedback` or `End Demo`. Localization keeps the visible titles translated; routing IDs remain English. Its output remains `Topic.feedbackForm`. The previous JSON-card version is retained in `backups/form7.FormSubmission.JSON.yaml`.

Condition `conditionItem_bmmUxX` checks only `Lower(Trim(Topic.feedbackForm)) = "start feedback"` and redirects at `C6mX0V` to `brandon_Agent09232026.topic.wf9dev.Feedback`. Confirm that this is the actual schema name of the topic displayed as helper4.Feedback in your agent.

`endDemoSelected` recognizes `End Demo` and clears feedback. An unknown output repeats the card and displays the actual value instead of treating it as a confirmed End Demo. `Tk0LIe` ends the current topic and returns to InternationalFormWorkflow, whose existing final node ends its topic stack.

## helper4.Feedback

Use a normal `OnRedirect` trigger and remove `startBehavior: CancelOtherTopics` so this topic can be called as part of the existing form. Keep question `CF7xoW`, the English `skip` check, and thank-you message `AoIk6Y`.

Replace the Fallback redirect `U7pelG` with `EndDialog` (`returnFromFeedback`). It returns to FormSubmission. The old redirect could reenter the no-answer/form-entry path after feedback; the actual behavior depends on the live Fallback topic, which was not supplied.

Microsoft documents that End current topic returns to its calling topic. [Manage topics](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

## Test before the next change

1. Proceed -> Start Feedback -> enter a comment: thank-you, then complete with no Fallback or new form.
2. Proceed -> Start Feedback -> type `skip`: empty Global.feedbackData, then complete.
3. Proceed -> End Demo: no feedback prompt; empty Global.feedbackData; complete.
4. Repeat in both previously tested languages with translated button titles. `debugFeedbackAction` displays `FeedbackAction=[...]` immediately after submission. The intended result is `Start Feedback` or `End Demo` regardless of the displayed language. Remove this temporary diagnostic after the live test passes. The text command `skip` is currently English, as in the supplied topic.

## Preserve translated titles; test formula-mode submission

The user clarified that visible button titles must remain translated. The earlier English-only localization change has been reversed in all six files. Localization keys, translated messages, and the two original translated button titles are retained.

The saved review card uses Power Fx and routes on English Proceed/Edit values, while its supplied localization files translate the visible titles. The feedback card previously used JSON. This revision changes only the feedback card's representation, preserves its data/output/conditions, and adds one temporary diagnostic. That is a controlled test of the user's reported JSON-versus-Power-Fx behavior. It is not proof that all Copilot Studio JSON cards mishandle action IDs, or a guarantee that Formula mode changes a host's submission payload.

In Studio, card `8NF3jZ` -> Properties -> Formula converts a JSON card to Power Fx. Alternatively, paste this revised YAML topic. Keep the output binding actionSubmitId -> Topic.feedbackForm with type String. If the English-only files were already uploaded, upload the restored translated files from `Localizations/` for each matching language. If Studio changes resource keys when saving the formula card, download the current localization export and reconcile the entries for this same topic/node.

Microsoft documents Formula mode and its built-in JSON conversion, and button titles as localizable Adaptive Card text. [Ask with Adaptive Cards](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-ask-with-adaptive-card), [Localize Adaptive Card content](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/localize-adaptive-cards).

The next backend step is explicit mapping and validation of the International Form Email inputs. The current submission remains a mock, and no email has been sent by these local changes.
