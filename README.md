# International Support Form

Snapshot saved on October 1, 2026 from the code supplied in this chat.

The supplied logic is preserved in `topics/` and `flows/`. Findings and suggested changes are separate; no fixes have been applied to this snapshot or to the live agent.

- [Current flow](CURRENT_FLOW.md): entry points, topic connections, variables, required fields, and backend flows.
- [Bug and logic review](BUG_REVIEW.md): prioritized findings with reproduction steps and suggested corrections.
- [AI-to-form handoff](AI_HANDOFF.md): a concrete design for answering first and offering the support form when needed.
- [Verification record](VERIFICATION.md): checks performed and remaining live-agent checks.
- [Corrected editor copy](revisions/helper2.FormValidationEditor.yaml): the latest supplied editor with normalization, TRACE A, and all five section conditions changed to use `Topic.SectionRoute`. This addresses section routing only; the other review findings remain.
- [Proceed and review-loop repair](revisions/review-loop/README.md): seven revised topic copies that remove nested review calls and give Proceed/Edit stable identifiers. This supersedes the earlier editor-only revision when applied as a set.
- [Feedback completion repair](revisions/feedback-completion/README.md): revised submission and feedback topics with a Power Fx submission card, English routing IDs, translated button titles, and a normal return after feedback.
- [Localization upload notes](Localizations/README.md): six supplied localization files with translated feedback button titles restored.

## Saved files

`topics/` contains 15 YAML topic snippets. Filename labels follow the names supplied in the chat. Internal dialog identifiers, including `LanguagePreferenceCopy`, `wf2` through `wf8`, and `wf9dev.Feedback`, are preserved.

`flows/InternationalFormEmail/` contains the trigger, email action, and response as three individual JSON files. `flows/InternationalFormTranslation/` contains the trigger, two translation actions, and response as four individual JSON files. JSON formatting is normalized; property values and expressions are preserved, including the leading space in ` writtenFeedback`.

These files are source snippets, not a complete importable Power Platform solution. Six secondary-language localization files were supplied later and are saved in `Localizations/`. Agent settings, connections, global-variable definitions, action wrappers, and several referenced system topics were not supplied.

The source code fences originally labeled bash, arduino, or css were YAML and are saved with `.yaml` extensions. The review card remains a Power Fx expression embedded inside YAML.
