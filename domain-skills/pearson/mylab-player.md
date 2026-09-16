# Pearson MyLab homework player

The course shell at `mylabmastering.pearson.com/courses/<course>/menu/<menu>` embeds pages from `mylab.pearson.com`. A homework overview uses `/Student/OverviewHomework.aspx?homeworkId=<id>`; the player uses `/Student/PlayerHomework.aspx?homeworkId=<id>&questionId=<number>`. Treat assignment identifiers and execution-context identifiers as session data; rediscover them each run.

## Navigation and verification

- The question player is in a cross-origin iframe. Its main-world context exposes a legacy Dojo/Dijit application. Top-document JavaScript cannot read the inner question.
- Completed parts remain in the DOM with disabled inputs. For current-part inspection, filter inputs by `!disabled`; equation-editor containers use `.eqEditor.disabled` for prior parts.
- The question list uses `a.questionLabel`. The player may ignore a navigation click during a transition. Wait for the header to identify the intended question before continuing.
- Feedback buttons are `OK` for another part and `Next question` after the final part. Some headers undercount the DOM's explanatory part headings. Follow feedback rather than counting all `Part` text nodes.
- Checking feedback is asynchronous. Verify the success message and current header before advancing. Do not resubmit just because a short wait showed no feedback.
- `Save` closes the player and returns to the overview. The prior execution context is then invalid. Reattach to the remaining course tab.
- The overview can display a stale aggregate score while its per-question statuses have already updated. Open the course Results page to verify the recorded assignment score.

## Legacy widgets

Prefer visible UI actions and screenshot verification. When a legacy control requires an inspection fallback, `.xlFillinItem` holds dropdown widget IDs and `.eqEditor` holds equation-editor IDs; `dijit.byId(id)` retrieves the widget. IDs can change between questions.

- Equation-editor widgets have `getEqText`, `clear`, `setEqText`, and `setChanged`. A value update needs the change notification before the checker enables.
- Dropdown widgets lazily construct their menu with `loadDropDown`. Menu children contain a visual and spoken math representation, so raw `innerText` may contain duplicated labels. Inspect `containerNode.innerText` or the menu's label options before matching a choice.
- Setting a radio's framework property can change its appearance without triggering the player's answer-change handling. A compositor click reliably triggers that handling.
- Wrap inspection JavaScript declarations in an IIFE. Repeated top-level `let` declarations in the same execution context can fail with lexical redeclaration errors.

## Graph inspection

The interactive graph widget is a `.graphControl` Dijit widget with a `_graphManager`. `getGraphObjectsData()` returns serializable editable-object data. Label collections may contain circular widget references and cannot be JSON-stringified. Static plotted series are separate graph parts with `xPts` and `yPts`; clearing editable objects does not remove those reference lines.

Graph labels are widgets, not strings. Do not pass plain text to an object's `setObjLabel`. Inspect the existing label widget's `getLabelOptions()` and use the exact option text. Labels can use markup such as `AE@sub{1}`, and the available label may differ from the wording used in the prompt.

These mechanics are UI details only. Determine the user's authorized scope and whether the activity is practice or graded before making submissions.
