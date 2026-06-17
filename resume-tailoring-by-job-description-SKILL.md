---
name: resume-tailoring
description: Use when the user wants to build a comprehensive master record of their career history and/or generate a resume tailored to a specific job description. Covers two workflows: (1) building or updating a master career corpus through document ingestion plus conversational interview, and (2) generating a job-specific resume from that corpus matched to a job description's requirements and language. Trigger on phrases like "build my resume corpus," "tailor my resume to this job," "here's a job description, make me a resume," or when the user uploads old resumes/CVs and wants to consolidate them.
---

# Resume Tailoring Skill

## Overview

This skill has two phases that can be run independently:

1. **Corpus Building** — ingest the user's existing resumes/CVs (Word, PDF, plain text, whatever they have), then interview them role by role to surface accomplishments, scope, and metrics that never made it onto paper. The output is one master corpus file: the most complete record of their career that exists anywhere.

2. **Tailored Generation** — given a job description (pasted text or uploaded file), mine the corpus for the most relevant material, write a resume in the job's own language where it's truthfully applicable, and produce clean Markdown + Word output.

**Core principle:** Truth-preserving optimization. The corpus only ever contains things the user actually did. Tailoring reorders, reframes, and re-emphasizes — it never invents.

**Mission:** A person's ability to get a job should depend on their experience, not on their resume-writing skill or their memory of their own accomplishments in an interview.

## When to Use Each Phase

**Run Corpus Building when:**
- No master corpus file exists yet for this user
- The user explicitly asks to build, rebuild, or update their corpus
- The user uploads old resumes and wants them consolidated
- It's been a long time since the last update and they mention a new role, promotion, or major project

**Run Tailored Generation when:**
- A master corpus already exists and the user provides a job description
- The user asks to "tailor my resume for X" with a JD available

**If neither exists yet** (no corpus, and a JD is provided): run Corpus Building first, then immediately move into Tailored Generation using the JD already in hand — no need to ask the user to re-paste it.

## Phase 1: Corpus Building

### Step 1.1 — Locate and ingest existing material

Ask the user where their existing resumes/CVs live if not already provided (uploaded files, a folder path, or pasted text). Files may be `.docx`, `.pdf`, `.doc`, or plain text/Markdown.

- For `.docx`: use the **docx** skill's text extraction (`extract-text`).
- For `.pdf`: use the **pdf-reading** skill's extraction approach.
- For `.doc` (legacy): convert to `.docx` first per the docx skill, then extract.
- For plain text/Markdown: read directly.

If the user has multiple old resumes (e.g., versions tailored for different past applications), ingest all of them — they often contain different bullets for the same role, and the goal here is the union of everything, not just the most recent version.

### Step 1.2 — Parse into a draft structure

Organize extracted content into a single working structure, grouped by role:

```markdown
## [Company Name] — [Title]
**Dates:** [Start] – [End]

### As written:
- [bullet 1]
- [bullet 2]
...

### Skills/tools mentioned:
[list]
```

Do this silently — it's an intermediate working draft, not something to show the user yet. Note internally which roles look thin (1-2 generic bullets) versus well-documented, since that shapes how much interview time each role needs.

### Step 1.3 — Expansion interview

This is the most important step. Old resumes almost always understate what someone actually did — bullets get cut for space, older roles get compressed, and people forget to mention things that felt routine at the time but are genuinely impressive (mentoring, budget ownership, crisis response, things they built that outlasted them).

Go through the roles **one at a time**, most recent first unless the user prefers otherwise. For each role, ask a small set of targeted questions rather than one giant generic prompt. Good question types:

- **Scope/scale:** "How many people/systems/dollars were involved in this?" "Who did you report to, and who reported to you?"
- **Metrics:** "Do you have a number for this — percentage, dollar amount, time saved, headcount?" If the user doesn't have an exact figure, ask for a reasonable estimate they're comfortable defending rather than leaving it vague.
- **Tools/methods:** "What specific tools, frameworks, or methodologies did you use day to day?"
- **Standout moments:** "Was there a specific project, crisis, or initiative in this role you're most proud of that isn't reflected in these bullets?"
- **Leadership/influence:** "Did you mentor, train, or influence people outside your direct chain?"
- **Continuity:** "Is there anything from this role that influenced how you operate now, or that you'd want a future employer to know about even if it doesn't sound like a typical bullet?"

Keep this conversational — a few questions per role, not an interrogation. Use `ask_user_input_v0` for quick scoped questions where multiple-choice genuinely fits (e.g., "roughly how big was the team you led?"), but most of these answers will be free text, so plain conversation works better than forcing everything into buttons.

For thin/older roles, it's fine to ask fewer, lighter questions. For roles directly relevant to the user's current career direction, go deeper.

After each role, briefly read back what was captured ("Got it — so at [Company] you also owned X and drove Y, with Z outcome") so the user can correct anything before moving on.

### Step 1.4 — Capture cross-role material

After going through individual roles, ask about anything that doesn't map cleanly to one job:
- Certifications, clearances, credentials, and when obtained
- Education (degrees, institutions, dates, honors)
- Publications, speaking engagements, patents
- Volunteer work, side projects, military service
- Awards and recognitions
- Technical skills and proficiency level, if relevant to their field

### Step 1.5 — Write the master corpus file

Consolidate everything into a single Markdown file: `master_resume_corpus.md`. This is the permanent source of truth and should be far more complete than any single resume — it's meant to be mined, not sent out as-is. Structure:

```markdown
# Master Career Corpus — [Name]
*Last updated: [date]*

## Professional Summary
[2-3 sentence overview of career arc, written plainly — not yet tailored]

## Experience

### [Company] — [Title]
**Dates:** [Start–End] | **Location:** [if relevant]

**Context:** [1-2 sentences on team size, scope, reporting structure]

**Accomplishments:**
- [Expanded bullet with metric where possible]
- [...]

**Tools/Methods:** [list]

[Repeat per role, most recent first]

## Education
[...]

## Certifications & Credentials
[...]

## Skills
[Grouped logically for the user's field]

## Other
[Publications, awards, volunteer work, side projects, etc.]
```

Save this file in the user's working directory (or wherever they indicate) and tell them plainly where it is. This file should be edited/appended to over time, not regenerated from scratch on every update — if a corpus already exists, Step 1 of any future session should load it and treat new information as additions, not a replacement.

**Confirm with the user before finishing this phase:** show a summary of what was captured (role count, rough bullet count, notable additions from the interview) and ask if anything important is missing before moving on.

## Phase 2: Tailored Generation

### Step 2.1 — Get the job description

Accept it as pasted text, an uploaded file (use the appropriate ingestion approach — docx/pdf skills if it's a file), or a URL (fetch it). If the corpus doesn't exist yet, run Phase 1 first.

### Step 2.2 — Deconstruct the JD

Treat the job description as a dataset, not prose to skim. Extract, explicitly and separately:

- **Core keywords:** hard skills, tools, software, methodologies, or certifications that appear multiple times or are clearly emphasized. Note the *exact phrasing* used — if the posting writes "Project Management Professional (PMP)," record that full phrase, not just "PMP."
- **The "Why" (immediate pain points):** the first 3-5 bullets under responsibilities almost always reveal what the hiring manager is actually hurting from right now. Flag these specifically — they outweigh later, more generic bullets when it comes to prioritizing what to lead with.
- **Required vs. preferred qualifications:** must-haves vs. nice-to-haves, kept distinct since they get treated differently in matching.
- **Soft skills / cultural-fit language:** the verbs and adjectives describing the ideal candidate — "collaborative," "autonomous," "fast-paced," "data-driven," etc. These matter for tone-matching the summary and competencies section, not just bullet content.

### Step 2.3 — Match corpus to JD

Go through the master corpus and identify, for each JD requirement, the strongest supporting material. Prioritize:
1. Direct, strong matches with metrics
2. Adjacent/transferable experience that can be honestly framed to the requirement
3. Gaps — note these internally; don't pad with weak material to fill a gap

Also identify, across the whole corpus, which entries skew toward the JD's actual emphasis. If a past role was genuinely 70% technical execution and 30% strategy, but the target job is heavily strategy-focused, that role's strategy-flavored accomplishments are what get promoted to its top bullets in the tailored output — the technical bullets for that same role get deprioritized or cut, not because they're untrue, but because they don't serve this narrative.

**Light-touch gap check:** if a required qualification has no corresponding material anywhere in the corpus, ask the user one targeted question about it before generating ("This role asks for experience with X — do you have anything relevant we haven't captured?"). Don't re-run the full interview. If there's no gap, skip straight to generation. Per the integrity principle below, a real gap should be addressed by emphasizing adjacent strengths and demonstrated agility — never by inventing coverage.

### Step 2.4 — Write the tailored resume

**Mirror the JD's vocabulary strategically, not mechanically.** Where the corpus genuinely supports a claim, use the JD's own terms rather than a synonym — if the posting says "Cross-Functional Leadership," write that, not "Led Multi-Departmental Teams," when describing the same real accomplishment. Match acronyms and phrasing exactly as the posting renders them. This is alignment, not keyword stuffing: each mirrored term should land inside a real, substantiated bullet, never as a disconnected skills dump at the bottom built just to catch an ATS scan.

**Rewrite the professional summary to speak to this role's core mission.** 3-4 lines, using the job title (or a close equivalent the corpus can honestly support) in the first sentence. This is the highest-value real estate in the document — it should read like it was written for this job, not adapted from a template.

**Build a competencies section ordered by JD relevance.** Pull from the corpus's Skills section, but reorder so whatever the JD emphasizes most appears first, not in whatever order the corpus happens to list it.

**Apply the X-Y-Z formula to bullets:** accomplished [X], as measured by [Y], by doing [Z]. Lead with the outcome, not the activity. Quantify wherever the corpus has a number — an unquantified bullet is a weaker bullet by default.

**Reorder bullets within each role by relevance to this JD**, not chronologically or by the order they appear in the corpus. The pain-points identified in Step 2.2 should map to whatever appears first under the most relevant role.

**Reframe titles/summaries honestly, not deceptively.** A title can be clarified ("Technical Program Manager" → "Program Manager, Environmental Systems") if the underlying scope genuinely supports it, but never changed to claim a role the person didn't hold.

**Cut what's irrelevant to this JD.** The corpus is comprehensive; the resume should not be. Aim for the length appropriate to the user's seniority and field (commonly 1-2 pages).

**Format for the 6-second scan.** Recruiters skim in an F-pattern, across the top, then down the left margin. Use standard, parseable headers (`## Professional Experience`, `## Education`, etc.) for both human and ATS navigation. Bold key metrics, critical tools, and major initiatives within bullets so they catch the eye on a skim, but bold sparingly enough that emphasis still means something; if every line has a bolded phrase, nothing stands out.

**No em-dashes in generated resume text.** Em-dashes (—) read as an AI-generated tell to many recruiters and ATS reviewers. Never use them in resume output, including the professional summary, bullets, and any other generated text. Use a period, comma, semicolon, or parentheses instead, whichever reads most naturally for that sentence. This applies to resume deliverables specifically; it doesn't change how Claude writes elsewhere in the conversation.

**Integrity check, every time:** tailoring is amplification, not fabrication. If the corpus only supports 60% of what the JD asks for, the job is to emphasize that 60% hard and let the resume's overall caliber imply the agility to pick up the rest — never to invent the missing 40%. If a claim in the tailored draft can't be traced back to something in the corpus, cut it before generating output.

### Step 2.5 — Generate output

Produce:
1. **Markdown** version of the tailored resume
2. **Word (.docx)** version, using the docx skill's document-creation guidance (Arial font, proper margins, real bullet formatting via numbering config — never manual bullet characters, US Letter page size set explicitly, bold runs for the emphasis called out in Step 2.4)

Save both to the outputs directory, but do not present them yet. Hold both files for the judge step below.

### Step 2.6 — Judge the output before presenting it

Before showing the resume to the user, run a separate verification pass. This is a distinct mental mode from generation: while Step 2.4 was optimizing for fit and persuasiveness, this step's only job is to find problems, not to defend or improve the draft. Treat the draft adversarially, as if reviewing someone else's work with no investment in it looking good.

Check two genuinely different things, in order:

**A. Factual accuracy against the corpus.** Go through the generated resume line by line. For every concrete claim (a metric, a tool, a team size, a title, an outcome, a quoted phrase), locate the specific sentence in `master_resume_corpus.md` that supports it. For each claim, classify it as:
- **Directly supported:** the corpus states this fact plainly. No issue.
- **Reasonably inferred:** the resume draws an honest, labeled connection or analogy from real corpus material (e.g., framing a military methodology as conceptually similar to a civilian one) but doesn't claim the corpus fact itself changed. Acceptable, but flag it so the user can see where interpretation entered the document.
- **Unsupported or overstated:** the resume states something the corpus doesn't contain, inflates a number, upgrades a "preferred" framing to sound like a requirement was fully met, or implies a credential, scope, or outcome that isn't in the corpus. This is a hard fail. It must be fixed before the document is shown to the user, not just flagged.

Also check the inverse: did the JD-mirroring in Step 2.4 go far enough to use the JD's exact terms, or did a "strategic mirroring" turn into a synonym substitution that quietly weakens the match? This isn't a fabrication risk, but it is a quality miss worth flagging.

**B. Document integrity of the generated .docx.** Convert the generated docx to PDF and rasterize it for visual inspection (the same pattern used during generation: LibreOffice conversion, then `pdftoppm`, then `view` each page as an image). Check for:
- Truncated or cut-off text, overlapping elements, or content pushed off the page
- Bullets rendering correctly (real bullet glyphs from the numbering config, not literal asterisks or dashes leaking through)
- Bold emphasis appearing where intended and nowhere else
- Headers present and properly styled, consistent spacing, no orphaned single lines stranded awkwardly across a page break
- Page count reasonable for the content and seniority level (flag if it ballooned past what Step 2.4's length guidance intended)
- The text content of the docx matches the text content of the markdown version (no silent drift introduced when porting from one to the other)

**Resolution:**
- If A finds a hard fail (unsupported/overstated claim): fix it directly, by cutting the claim, softening it to what the corpus actually supports, or pulling a real corpus fact to replace it, then regenerate the affected output and re-run both checks on the corrected version. Do not present a draft with a known fabrication, even temporarily, even with a caveat.
- If A finds only "reasonably inferred" items: these can ship, but mention them briefly to the user when presenting the resume so they know which lines involve interpretation versus stated fact, since they may be asked about that distinction in an interview.
- If B finds a rendering problem: fix the docx generation script and regenerate; re-run the visual check before presenting.
- If both checks pass clean: proceed to presenting the files with no extra caveats needed.

This step should be quick and silent from the user's perspective (don't narrate the whole checklist to them), but the fixing and re-checking must actually happen, not just be asserted. If a fix was needed, briefly mention what was caught and corrected when presenting the final resume, so the user has visibility into the judge step doing real work rather than it being invisible.

### Step 2.7 — Offer to update the corpus

If the interview process or JD matching surfaced anything genuinely new (a metric the user clarified, a project they hadn't mentioned before), ask if it's fine to fold that into the master corpus file so future tailoring benefits from it too.

## Notes on Sensitive Content

The corpus should capture everything the user tells you, including sensitive background (security clearances, prior government/military/intelligence roles, etc.) — completeness is the point of the corpus. Whether any given piece of background appears on a specific tailored resume is the user's call per application; don't apply blanket redaction rules unless the user states one for that specific application. If the user has indicated elsewhere that certain details (e.g., specific agency or role titles) are handled carefully on public platforms like LinkedIn, that doesn't automatically extend to resumes — ask if unsure rather than assuming.
