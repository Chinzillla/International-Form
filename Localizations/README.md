# Translated feedback button titles

Updated October 1, 2026 after the user clarified that feedback buttons must remain translated. The previous English-only replacement has been reversed for the two button-title entries for `brandon_Agent09232026.topic.wf8.FormSubmission`, card `8NF3jZ`:

| Entry suffix | Source title (translated in each file) |
| --- | --- |
| `.Card.body[5].actions[0].title` | `Start Feedback` |
| `.Card.body[5].actions[1].title` | `End Demo` |

The complete shared key prefix is `'dialog(brandon_Agent09232026.topic.wf8.FormSubmission)'.'trigger(main)'.'action(8NF3jZ)'`.

The original language-specific title values are restored. All other translations and resource keys are retained. The feedback topic is now proposed as a Power Fx card, following the working validation topic's format, while retaining English submit IDs and translated display text.

## Files to upload

| Language | Localization file |
| --- | --- |
| German | [localizations.de-DE.json](localizations.de-DE.json) |
| Spanish | [localizations.es-US.json](localizations.es-US.json) |
| Japanese | [localizations.ja-JP.json](localizations.ja-JP.json) |
| Korean | [localizations.ko-KR.json](localizations.ko-KR.json) |
| Simplified Chinese | [localizations.zh-CN.json](localizations.zh-CN.json) |
| Traditional Chinese | [localizations.zh-TW.json](localizations.zh-TW.json) |

In Copilot Studio, use Settings -> Languages -> Upload for each matching language and select its JSON file. These local changes have not been uploaded or published by Codex. Reset the test conversation after uploading and test Start Feedback and End Demo in each language.

Use these resources with the formula-card topic and two English routing conditions in `revisions/feedback-completion/form7.FormSubmission.yaml`. A localization file changes displayed text; it does not install those topic-code changes. If Studio's current export changes the card resource paths after conversion, reconcile these translations with that export before uploading.

## Adding a language

Translate button titles through the normal localization workflow. Keep submit IDs unchanged in the topic. The intended result is that the Formula card returns the English submit ID even while displaying a translated title. This must be confirmed in Studio; it has not been executed by Codex. The working validation topic and its localization entries were not changed.
