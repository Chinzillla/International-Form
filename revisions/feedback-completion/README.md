# Translated feedback buttons with fixed routing conditions

Apply these proposed topic replacements after the working review-loop changes. Originals in topics/ remain unchanged. No email or translation flow is added in this step.

## Confirmed incoming response

The user captured Spanish incoming payloads with actionSubmitId and altText both equal to the translated button title: Finalizar demostración or Iniciar comentarios. The bound Topic.feedbackForm matches those values. The response itself already contains a translated identifier; the output binding is not where an English ID becomes translated.

Conversion from JSON to Power Fx alone did not resolve this path. The layer generating these values and the reason FormValidation behaves differently remain unconfirmed.

## Current form7.FormSubmission change

The buttons retain English id and data.actionSubmitId values, but their titles now use localizable text variables:

| Node | Variable | Primary-language value |
| --- | --- | --- |
| setFeedbackStartLabel (SetTextVariable) | Topic.FeedbackStartLabel | Start Feedback |
| setFeedbackEndLabel (SetTextVariable) | Topic.FeedbackEndLabel | End Demo |

Both nodes run before card 8NF3jZ. The formula card uses title: Topic.FeedbackStartLabel and title: Topic.FeedbackEndLabel, so the exact displayed titles are available to the subsequent routing logic.

The output binding remains actionSubmitId -> Topic.feedbackForm (String). After the card:

1. captureRawFeedbackAction preserves the received value in Topic.RawFeedbackAction.
2. normalizeFeedbackAction recognizes either the English value or the current label variable and writes Start Feedback or End Demo into Topic.feedbackForm. It retains unknown raw values so the existing error path can show them.
3. Temporary message debugFeedbackNormalizedV3 shows the raw value, normalized value, and both current labels.
4. conditionItem_bmmUxX and endDemoSelected keep their two English conditions. No list of translated titles is needed.

For a Spanish label variable equal to Iniciar comentarios, a response containing that same text normalizes to Start Feedback. Adding a language only translates the label variables through the ordinary localization process; it does not add conditions.

This normalizes the observed title-based response. It does not change what the client sends or explain the platform's ID-versus-title behavior.

## Install the topic and localization entries together

1. Replace the live FormSubmission topic with form7.FormSubmission.yaml. Its internal name must be brandon_Agent09232026.topic.wf8.FormSubmission, called by InternationalFormWorkflow node taTAMt.
2. Save the topic and run Topic checker for the text-variable nodes, formula card, output schema, and normalization formula.
3. Upload each matching JSON file from Localizations/. The existing translations for the two button titles were moved to the new text-variable .Value entries; all other values are retained. If Studio's freshly downloaded export uses different paths, reconcile the two entries with that export before upload.
4. Reset the conversation and click the newest card.

Microsoft documents SetTextVariable for localizable text referenced by Adaptive Cards. [Multilingual agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/multilingual), [Localize Adaptive Card content](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/localize-adaptive-cards).

The previous JSON topic is in backups/form7.FormSubmission.JSON.yaml. Localization files immediately before this key migration are in backups/localizations-before-label-variables/.

## helper4.Feedback

Use a normal OnRedirect trigger and remove startBehavior: CancelOtherTopics so this topic can return to its caller. Keep question CF7xoW, the English skip check, and thank-you message AoIk6Y.

Replace the Fallback redirect U7pelG with EndDialog returnFromFeedback. This returns to FormSubmission, which returns to InternationalFormWorkflow. [Microsoft topic management documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/authoring-topic-management).

Confirm the actual Feedback topic has internal name brandon_Agent09232026.topic.wf9dev.Feedback, as called by C6mX0V.

## Live tests still required

1. English Start Feedback: Routed = Start Feedback; collect a comment and finish normally.
2. Spanish or German translated Start Feedback: Raw matches StartLabel; Routed = Start Feedback; open Feedback.
3. Translated End Demo: Raw matches EndLabel; Routed = End Demo; complete without a feedback prompt.
4. Type skip in Feedback: clear the feedback and return normally. This command is still English, as in the supplied topic.
5. An unknown response: keep the existing retry/error path rather than treating it as End Demo.

After these pass, remove debugFeedbackNormalizedV3. The raw capture and normalization nodes remain necessary for this compatibility route. Codex has not executed or type-checked this revised topic in Studio.
