# Feedback button titles kept in English

Updated October 1, 2026 for the user's requested routing pattern. In all six supplied localization files, only the two button-title entries for `brandon_Agent09232026.topic.wf8.FormSubmission`, card `8NF3jZ`, are changed:

| Entry suffix | Value |
| --- | --- |
| `.Card.body[5].actions[0].title` | `Start Feedback` |
| `.Card.body[5].actions[1].title` | `End Demo` |

The complete shared key prefix is `'dialog(brandon_Agent09232026.topic.wf8.FormSubmission)'.'trigger(main)'.'action(8NF3jZ)'`.

All six files parse as valid JSON. Comparison against the repository version confirmed exactly two changed values per file, with all keys and other values preserved.

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

Use these resources with the matching button title/id/data values and two conditions in `revisions/feedback-completion/form7.FormSubmission.yaml`. A localization file changes displayed text; it does not install those topic-code changes.

## Adding a language

Keep these two title values as `Start Feedback` and `End Demo` during translation. Routing conditions need no additional language aliases. All other translations and resource keys were retained. The working validation topic and its localization entries were not changed.
