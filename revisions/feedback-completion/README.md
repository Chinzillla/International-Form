# Return normally after mock submission feedback

Apply these two proposed replacements after the working review-loop changes. Originals in `topics/` remain unchanged. No email or translation action is added in this step.

## form7.FormSubmission

At the beginning of this topic, `initializeFeedbackData` clears feedback for this form. Card `8NF3jZ` uses the same title, id, and data.actionSubmitId for each button: `Start Feedback` and `End Demo`. Its output remains `Topic.feedbackForm`.

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
4. Repeat in both previously tested languages after keeping the two button-title localization entries in English, as described below. The text command `skip` is currently English, as in the supplied topic.

## Keep the button titles equal to the English IDs

At the user's request, both feedback buttons retain English titles in all languages, using the same values for title, id, and data.actionSubmitId. Matching these three source values does not itself prevent localization from translating the titles.

For each secondary language, go to Settings -> Languages -> Upload -> download the current JSON or ResX localization file. Locate the two button-title entries for form7.FormSubmission card `8NF3jZ` (the actions inside the ActionSet in body[5]). Keep their values as `Start Feedback` and `End Demo`, and upload the file. Other displayed content can use its normal translations. When adding a language, leave these two exported title values in English. No new routing conditions are needed.

Microsoft documents button titles as localizable static Adaptive Card text. [Localize Adaptive Card content](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/localize-adaptive-cards).

The proposed localized-label-variable alternative was withdrawn after the user clarified that these two titles should remain English, following the working review-button pattern.

The next backend step is explicit mapping and validation of the International Form Email inputs. The current submission remains a mock, and no email has been sent by these local changes.
