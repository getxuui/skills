---
name: present
description: Shows the React UI you're working on in xuui, so the person can compare variants side by side and comment on them. Use when someone types /present or asks to show, compare or review UI in xuui.
---

# Present in xuui

Turn the work in the current conversation into a running, reviewable xuui study. The handoff is a verified local URL, a short explanation of the design question, and the differences worth reviewing. After the handoff, keep listening for the person's review.

## Stay in xuui

xuui is the presentation surface for this work. Once a design is in xuui, keep presenting it there.

- When another skill or workflow is also in play, use its method for choosing directions, but build the directions as xuui variants. Never build its picker, a switcher inside the application, or a separate preview page.
- "Show it in the whole page", "in the site" or "in context" means a larger xuui study: render the surrounding page or layout around the design, with the same variants.
- Change the real application only when the person asks to implement or ship a direction. Keep the study, so the choice stays reviewable.

## Resolve what to present

Use the requested UI, reference, or recent work as the brief. Present an existing implementation faithfully. When the brief asks for alternatives, make their differences answer the design question; choose the number from the brief and the useful comparisons. Ask only when the conversation and workspace leave the subject unclear.

Find the study workspace from the current project: an installed workspace has `xuui.studies.ts` and a local xuui dependency. Read that version's README, `docs/authoring.md`, and exported types, plus a nearby registration. Use the installed CLI rather than downloading a newer alpha into an existing workspace. A xuui source checkout instead uses `examples/studies.ts`, `src/xuui/model.ts`, and `src/xuui/parameters.ts`; read `CONTEXT.md` there. Do not infer the consumer workspace by walking upward from an installed skill.

The alpha presents registered React components in a standalone workspace. If a new workspace is needed, run `npx --package @getxuui/xuui@0.1.0-alpha.5 xuui init <directory>`, then use its pinned installation. For work from another codebase, inspect its tokens, neighboring UI, responsive constraints, providers, and data. Adapt the relevant component and necessary dependencies without silently redesigning it. Include a context wrapper when isolation would distort the review, and disclose mocked data or unavailable backend behavior.

## Frame an exploration

When alternatives are requested, state the design question, name the primary variable, and say what stays constant. Plan directions that answer that question. Start them with matched content, meaningful state, viewport width, and scale. Do not generate alternatives for a request to present an existing design faithfully. Let the brief and useful comparisons determine the number.

## Register the study

### Announce a new variant

Before building a new variant in an existing study, register the in-progress direction with a stable ID and a meaningful name describing the design, such as "Paper, refined". Use that name from the start and retain it when registering the finished component. When `xuui_report_variant_progress` is available, call it with `status: "working"`, `studyId`, `variantId`, the required `name`, and the parent variant IDs in `from`. Optional `width` and `height` set the loading artboard's size in CSS pixels. The reviewer sees the named direction with a small spinner in the sidebar and above its dot-grid artboard in All, while keeping their current view and comments.

Without that MCP tool, check whether the installed CLI's `--help` lists `progress`. If it does, run from the study workspace:

```sh
npx --no-install xuui progress --study <study-id> --variant <new-variant-id> --name "Refined spacing" --from <parent-variant-id>
```

Repeat `--from` for multiple parents; omit it for an independent direction. Use `--width` and `--height` when the design's dimensions are known. In a source checkout, use `node src/cli.ts progress` with the same arguments. Older installations without this capability can continue presenting variants; say that live progress is unavailable rather than upgrading the workspace silently.

Register the finished component with the **same variant ID** and parents. It replaces the loading artboard automatically. After verifying it, report `status: "done"` with `studyId` and `variantId`, or run `npx --no-install xuui progress --study <study-id> --variant <new-variant-id> --done`. Clear the signal the same way when abandoning a variant. Let the workspace apply updates in place; preserve the reviewer's browser session and comment drafts.

### Add the registration

Put the design and its adapter beside the study's registration, following the existing studies. Add the study to the central registry, or extend the relevant existing study when the work explores the same question.

- Use the product terms **study**, **variant**, **canvas**, **layer**, and **guide**. Register a study with `defineStudy` and list its `variants`.
- Give each variant a stable, descriptive ID unique across all registered studies. Add a new ID when advancing a direction that should remain reviewable. Preserve the earlier implementation; sharing a mutable component does not freeze its history.
- When the installed version supports history (alpha.3 onward), set `from` to the parent variant IDs and `answers` to the submitted comment IDs the new variant addresses. The All view uses that lineage to arrange the study. Optional `labels` hold up to three short tags in the person's words, such as "Shipped"; xuui gives them no status or ranking. Follow the installed authoring docs and types for the exact shape.
- Write a useful name, a one-line description of at most 100 characters, and a single-paragraph rationale of one or two sentences (at most 280 characters). Explain what the variant tries and what it trades off. In alpha.5 onward, reviewers read both from the frame label.
- Register ordinary React components. xuui supplies navigation, panning, zoom, inspection, comparison, and comments; the study supplies the design and the providers or data it needs.
- Scope the study's CSS so it cannot restyle studio chrome or unrelated variants. Give the design meaningful dimensions; a percentage width without a sized containing block can collapse on the canvas.
- Add `data-xuui-layer-name` where the DOM alone gives an unhelpful layer name.
- When aligning a design by a grid, declare that grid on the variant if the installed types support `grids`. Use pixel spacing for a spacing system and column or row grids for layout alignment. Follow the installed authoring docs for custom size, count, gutter and margin fields, then verify the overlay matches the design.

## Expose states and controls

The sidebar shows the controls a variant declares, and the reviewer comments on whatever is rendered. Choose the kind of control that matches the design:

- **`machine`** for the design's distinct states and the events that move between them: empty, loading, loaded and error; closed and open; the steps of a form or flow. The reviewer can jump to any state from the sidebar and follow the transitions in Play mode, so each state can be judged and commented on. Name states and events the way the product does. Render from `machine.value`, and wire the design's real interactions to `machine.send(event)`, so clicking in Play mode follows the same machine.
- **`state`** for independent values the design reacts to, such as a favourited toggle or an item count.
- **`props`** for design parameters worth comparing, such as copy, radius or width.

When the design has meaningful states beyond the happy path, expose them with a machine rather than presenting one state. Don't invent states the design doesn't have. Expose only controls that change the rendered design, and check every state renders. See the installed `docs/authoring.md` for the exact shape.

Keep changes to the requested study and the registration needed to present it. A presentation request does not authorize redesigning the studio itself.

Done means the requested design renders from a registered variant and its exposed controls work.

## Run and verify

In an installed workspace, use `npx --no-install xuui dev`; in a source checkout, use its package scripts. Check for an existing preview first and reuse it when it serves the intended workspace. Leave unrelated processes alone. Keep the local server running for the reviewer and use the actual printed origin.

Open a URL using the actual server origin and the registered IDs:

- Focused variant: `/?study=<study-id>&variant=<variant-id>`
- Comparison: the same URL with `&view=all`

Verify the rendered variant identity, rather than treating an HTTP 200 as proof: unknown IDs fall back to an existing study. Inspect the UI, exercise important interactions in Play mode, and try each exposed control. Apply the same usability floor to each direction: keyboard access, visible whole-control focus, readable text, appropriate reduced motion, and realistic content, states, and widths. Run relevant project checks and report failures or unavailable checks accurately.

Use isolated test storage for submission checks. Leave the reviewer's real draft comments and submitted records intact.

Done means the link opens the intended design, meaningful interactions work, and the review boundary is clear.

## Hand off

Return the focused link, plus a comparison link when useful. Explain what to judge in a few sentences and name material limitations such as mocked data. The reviewer uses Play for interactions, C for comments, and Submit review for the handoff. End by saying you're listening for their review and they can keep chatting meanwhile.

For alternatives, add a short table of what changed, when each direction fits, and its main cost. State the reviewed width and state. Do not silently pick a winner. When asked to advance a choice, verify the promoted design in its source context and retain its reviewable study; preserve old implementations and add stable IDs for further comparable work.

## Listen for feedback

After the handoff, wait for the review without being asked.

- If the xuui MCP tools are available, call `xuui_wait_for_review` with the study's `studyId`. Each call returns within 50 seconds. Call it again with the returned `knownReviewIds` until a review arrives or the person asks you to stop.
- If they aren't available, the workspace's MCP server isn't connected. Offer to connect it, following the installed `docs/mcp.md`, and say if the host needs a restart to load it. Until then, if your host can run a background command that notifies you when it finishes, use one to wait for a new file in `.xuui/reviews/`. Otherwise, ask the person to tell you when they've submitted.

When a review arrives, read the complete package, then acknowledge it with `xuui_acknowledge_review`. A receipt means you read it, not that its requests are done. Browser drafts are not submitted feedback. Act on the comments: make changes as new variants with new IDs, verify them, hand off the new links, and listen again.
