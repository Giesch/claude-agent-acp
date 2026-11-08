# AskUserQuestion form contract

Status: Experimental

This document is the wire contract of the ACP form elicitation that `claude-agent-acp` sends for the Claude `AskUserQuestion` tool.
It applies to every client that declares form elicitation support.
Without form elicitation support, the adapter disables the tool when it creates the Claude session.

## Fields

The adapter sends one form for one tool call.
Each question at index `n` gets 2 properties:

| Key                   | Type                       | Content                                                |
| --------------------- | -------------------------- | ------------------------------------------------------ |
| `question_<n>`        | `string` with `oneOf`      | A single-select question.                              |
| `question_<n>`        | `array` with `items.anyOf` | A multi-select question.                               |
| `question_<n>_custom` | `string`                   | Free text for the same question. Its title is `Other`. |

- The `const` of each option is the option label.
- The option description is in the standard `description` field.
- No property is required. The user can leave any question blank.
- With 1 question, the question text is in `message`, and the field has no `description`.
- With 2 or more questions, each select field has the question text in `description`.

## Option preview

An option with a preview carries it under `_meta["_claude/askUserQuestionOption"]`:

```json
{ "preview": "<text shown when the option has focus>" }
```

## Custom answer companion

Each `question_<n>_custom` property carries `_meta["_claude/askUserQuestionCustomAnswer"]`:

```json
{ "questionId": "question_0", "withSelection": "note" }
```

- `questionId` names the select field of the same question.
- `withSelection` tells how the text combines with a selection.
  It is `"note"` for a single-select question.
  It is `"append"` for a multi-select question.

A client can render the companion as part of its select question.
A client that does not read the key can render the companion as an independent optional text field.

## Response semantics

The client sends the original property keys in the `accept` content.
The client sends both keys when both have a value.
The adapter trims the custom text.
The adapter ignores blank custom text.

| `withSelection` | Selection | Custom text | Result                                                                       |
| --------------- | --------- | ----------- | ---------------------------------------------------------------------------- |
| any             | none      | present     | The text is the answer.                                                      |
| any             | present   | none        | The selection is the answer.                                                 |
| `"note"`        | present   | present     | The selection is the answer. The text goes to `annotations[question].notes`. |
| `"append"`      | present   | present     | The text is one more answer item after the selected labels.                  |
| any             | none      | none        | The question has no answer.                                                  |

Do not clear the selection when the user types custom text.
Do not clear the custom text when the user selects an option.
For a multi-select question, do not join the text into the selection.
The adapter joins the items with the quoting rule of the Claude CLI.

## AIR

An AIR client also gets `_meta.jetbrains.air.customAnswer` and `_meta._askUserQuestionCustomAnswer` on each companion.
See [AIR extensions](air-extensions.md#question-custom-answers).
