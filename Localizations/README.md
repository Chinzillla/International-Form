# Translated feedback button titles

Updated October 1, 2026 to keep feedback buttons translated while normalizing their observed title-based responses. The two existing translated button-title values were moved to SetTextVariable entries in `brandon_Agent09232026.topic.wf8.FormSubmission`:

| Entry after the topic/trigger prefix | Source title (translated in each file) |
| --- | --- |
| `'action(setFeedbackStartLabel)'.Value` | `Start Feedback` |
| `'action(setFeedbackEndLabel)'.Value` | `End Demo` |

The complete shared key prefix is `'dialog(brandon_Agent09232026.topic.wf8.FormSubmission)'.'trigger(main)'.`.

All six files parse as valid JSON. Exactly two resource keys per file were moved from the static card-title paths to the text-variable paths. All translated values and all other keys are preserved. The pre-migration files are backed up under `revisions/feedback-completion/backups/localizations-before-label-variables/`.

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

Save the revised topic in `revisions/feedback-completion/form7.FormSubmission.yaml` before uploading these files. It uses the translated text variables as button titles and normalizes the returned title to the English routing value. A localization file does not install the topic-code changes. If Studio's current export uses different text-variable paths, reconcile these two entries with that export before uploading.

## Adding a language

Translate the two text-variable `.Value` entries through the normal localization workflow. The topic compares the returned value with its current translated label, then normalizes it to `Start Feedback` or `End Demo`. No additional condition is needed per language. The raw response can still contain a translated title; the canonical topic variable is what the two English conditions use. This has not yet been tested in Studio. The working validation topic and its localization entries were not changed.
