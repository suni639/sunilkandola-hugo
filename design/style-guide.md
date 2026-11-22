# Writing style guide

How articles on this site are written. The reference pages are the November 2026 ones:
`projects/house-layout-planner`, `projects/regulatory-horizon-scanner` and the
`projects/agentic-payments-lab` series. The June 2026 pieces (`projects/da-intel-01`,
`insights/agentic_ai_intel`) use an older style and are not the model.

## Tone

- First person, conversational and plain: a capable person explaining to a smart friend, not a whitepaper.
- Honest about limits and mistakes ("the AI was confident, and sometimes confidently wrong").
- Measured claims backed by numbers. No superlatives or hype ("100% free", "significantly sharper").
- A light personal touch where it fits, often in the closing line.
- Risk-minded: what could go wrong, and how would you know?
- "I" for solo work, not "we" (unless it genuinely was a joint effort, e.g. with family).

## Structure: project articles

1. `{{< lead >}}`: two or three sentences, the situation and what I did.
2. Disclaimer `{{< alert "circle-info" >}}` where needed (fictional data, not advice).
3. **The problem / Why I built it**: real-world context, explained for newcomers.
4. **What I built / How it works**: Mermaid diagram, then numbered steps with **bold lead-ins**.
5. **Worked example or case study**: one concrete case followed from start to finish.
6. **Results / findings**, often as a table.
7. Optional **Try it yourself** (alert plus numbered steps) or **Under the hood** (technology table).
8. **Key takeaways**: three or four numbered points with bold lead-ins, then a one-line closing sentence.

Separate sections with `---`. Headings are `##`, sentence case, short, sometimes a question.
Table of contents on.

## Structure: insight articles

- Same voice and principles, shorter (800 to 1,200 words).
- An argument, not a build log: lead with the claim, give two or three pieces of evidence
  (ideally from my own projects), a counterpoint or limits section, then a "so what".
- YAML front matter, to match the insights section, plus a `featureImage`.

## Format

- Page bundle: `content/<section>/<slug>/index.md` with images alongside, `feature.png` as the card image.
- Projects use TOML front matter: `title`, `date`, `draft`, `description` (search engines),
  `summary` (cards; a different sentence), `featureImage`, `featured`, `tags`,
  `categories = ["Projects"]`, `showAuthor`, `showTaxonomies`, `showReadingTime`, `showTableOfContents`.
- Titles: a hook plus the subject ("Will the Sofa Fit? Planning Our New Home in 3D with AI").
- Length: around 1,500 words for a standalone project. Longer series put technical detail in separate build notes.
- Visuals: `figure` with descriptive alt text and a one-line caption; `gallery` for side by side;
  Mermaid for flows; tables for comparisons.
- Link the live dashboard, repo or related pages early.

## Writing principles

- Plain English first. Bold and explain jargon on first use in one newcomer-friendly sentence. Expand acronyms on first use.
- British spelling (tokenised, programme, per cent).
- No em dashes. Use commas, colons or full stops.
- Short sentences and paragraphs, one idea each.
- Concrete over abstract: real figures, dates, names.
- Source key facts and link to the official source; recheck against official pages.
- No AI filler: "it is evident", "leverage", "seamless", "game-changer", "delve", "robust".
- Show the human role: what I knew or decided that the tool or model couldn't.
