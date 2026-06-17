# resume-tailoring-by-job-description
A Claude skill that turns your scattered resume history into one comprehensive career corpus, then generates job-specific resumes tailored to any job description, without inventing anything.

How it works
Phase 1 builds a master corpus from your existing resumes and CVs (Word, PDF, plain text, whatever you have), then interviews you role by role to surface accomplishments, metrics, and context that never made it onto paper. The result is one markdown file that's more complete than any resume you've ever sent out, meant to be mined, not shipped as-is.
Phase 2 takes a job description, deconstructs it into keywords, requirements, and the hiring manager's actual pain points, matches it against your corpus, and writes a tailored resume that mirrors the posting's own language where your real experience supports it. Output comes as both Markdown and a formatted Word document.
A built-in judge step checks the generated resume line by line against your corpus before showing it to you, flagging or fixing any claim that isn't actually supported, and visually verifies the Word document rendered correctly.
Core principle
Tailoring is amplification, not fabrication. The corpus only ever contains things you actually did. Generation reorders, reframes, and re-emphasizes real experience to fit a role; it never invents qualifications you don't have.
Setup
Drop SKILL.md into your Claude skills folder. No configuration needed; the first time you use it with no existing corpus, it'll walk you through building one.
