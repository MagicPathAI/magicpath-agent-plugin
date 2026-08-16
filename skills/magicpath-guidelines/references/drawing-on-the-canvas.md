# Drawing on the Canvas — Native Shapes vs. MagicPath Components

**Load this when** the user wants diagrams, flowcharts, wireframes, user journeys, sitemaps, sticky-note boards, callouts, or annotations on their MagicPath canvas — FigJam-style work — or when you need to decide between drawing native shapes and building a component.

## The decision boundary

MagicPath's primary content is the **component**: a real, interactive React prototype built through a code session. That is this plugin's default output, and nothing here changes it. Native canvas shapes are the **thinking layer**: fast, lightweight drawings for communicating structure, flow, and feedback. A component is a product; a drawing is a conversation about a product.

| The user asks for | Build with |
|---|---|
| A screen, page, UI, prototype, app, "design" — anything meant to look real or behave | **Component** (code session) |
| A flowchart, architecture or system diagram, user journey, sitemap, mind map | **Shapes** |
| A wireframe, lo-fi sketch, "boxes and arrows", layout exploration | **Shapes** |
| Sticky notes, brainstorm or retro board, labels, callouts | **Shapes** |
| Feedback or markup on existing designs ("circle what changes") | **Shapes**, drawn around the designs — never edits to them |
| "Wireframe it, then build it" | Shapes first; when the user approves the structure, a component |

Signals: fidelity words — *real, working, interactive, clickable, polished, styled* — mean a component. Communication words — *diagram, sketch, map, flow, wireframe, annotate, brainstorm* — mean shapes. When a request is genuinely ambiguous ("mock up a checkout flow"), default to a component (that is what MagicPath is for) and mention in your reply that a wireframe or flow diagram is available if they wanted the sketch instead. Do not stop to ask.

Shape work is its own direction: no code session, no working directory, no build. Never start a code session to produce a wireframe, and never deliver an arrangement of static shapes when the user wanted something interactive.

## Using the shape tools well

The server owns the mechanics — read the canvas-management section of its instructions and the tool descriptions for pacing, parent-relative coordinates, stacking, refusal rules, and its warnings about undo. The craft below is what the tools cannot decide for you.

- **Read before you draw.** Fetch the canvas context first: place new work in clear space, reuse existing frames, and target real `shapeId` values instead of guessing.
- **Structure first, content second.** Establish the container (a named frame) and regions before filling them; let the drawing build up visibly in logical steps.
- **Let the server do layout math.** Use the arrange capability for aligning, spacing, and grids; reserve exact coordinates for when the user gave them.
- **Group what belongs together** so the user can move or delete a diagram, legend, or annotation set as one thing.
- **Smallest change wins.** Never restyle, rearrange, or "tidy" shapes the request didn't name — these edits bypass the user's undo.

## Craft: diagrams

- Pick one flow direction — left-to-right or top-to-bottom — and keep it for the whole diagram.
- One idea per node; labels of a few words, not sentences. Sticky notes carry commentary; geo shapes carry structure.
- Label arrows at decision points (yes/no, success/failure). Diamonds for decisions, rectangles for steps, ellipses for start/end.
- Color is meaning, not decoration: neutral for the base, one accent for the path or state that matters. A legend when colors encode categories.
- Name regions with frames when a diagram has swimlanes or phases.

## Craft: wireframes

- Lo-fi on purpose: boxes, lines, text, nothing else. Grayscale base, at most one accent. A wireframe that looks finished invites feedback about the wrong things.
- One frame per screen, named like the screen ("Checkout — step 2"). Side-by-side frames for a flow, arrows showing the path between them.
- Show hierarchy with size and position, not styling. Label the regions ("Nav", "Hero", "Primary CTA") so the structure reads without explanation.
- An X-crossed box is a universally read image placeholder; lines of text shapes stand in for copy.
- When the user later asks to build a wireframed screen, treat the wireframe as the structural brief for the component — carry its regions and hierarchy over, then apply real design judgment (the Design Defaults) to the component itself.

## Craft: annotations

- Never modify the component or image being discussed — annotate beside and around it, arrows pointing at the exact spot.
- Sticky notes for comments, numbered when order matters; one color per reviewer or per theme if there are several.
- Group each annotation pass so it can be cleared in one action once addressed.

## Boundaries

- Components and images on the canvas can be moved, stacked, and grouped, but their appearance and content belong to the component and image tools — expect and respect refusals.
- Do not use shapes to imitate a finished UI, and do not screenshot-trace an existing component into shapes; if the user wants a visual copy, they want a component.
- Delete shapes only when asked.
