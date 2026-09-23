# AmelTech Claude Skills

A collection of [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) for Claude, built by **AmelTech** for engineering, science and academic work: rigorous problem solving and token-efficient reasoning.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Skills

| Skill | Status | What it does |
|---|---|---|
| [AmelTech Class](skills/ameltech-class/) | ✅ Available | Solves exam papers, lab manuals, derivations and proofs in math, physics and engineering; verifies every step by computation; detects and corrects errors; draws circuits, graphs and diagrams; delivers a compiled PDF plus the editable LaTeX (`.tex`) source. |
| [AmelTech Chat Efficiency](skills/ameltech-chat-efficiency/) | ✅ Available | Adaptive reasoning depth (direct answer, light council, or full seven-pillar council), redundancy removal without losing substance, delta follow-ups, and honest token telemetry (never fabricated). |
| AmelTech IEEE | 🚧 Coming soon | Coming soon. |
| AmelTech File | 🚧 Coming soon | Intelligent document reconstruction between PDF, DOCX, PPTX, LaTeX and images with OCR, handwriting and equation recognition. Will be released once all PPTX and DOCX generation issues are solved. |

## Repository structure

```text
AmelTech-Claude-Skills/
├── README.md
├── LICENSE
├── .gitignore
├── dist/                              Ready-to-upload skill packages
│   ├── ameltech-class.skill
│   └── ameltech-chat-efficiency.skill
└── skills/                            Source of each skill
    ├── ameltech-class/
    │   ├── SKILL.md
    │   ├── assets/        LaTeX template
    │   ├── references/    Documents, exam papers, lab manuals, figures, LaTeX rules, verification
    │   └── scripts/       build.py, check_tex.py, check_layout.py, fit.py, label_spots.py
    └── ameltech-chat-efficiency/
        ├── SKILL.md
        ├── README.md
        ├── references/    Reasoning council, token optimization, telemetry, workflow engine
        └── scripts/       telemetry.py
```

## Installation

### Claude.ai (web, desktop, mobile)

1. Download the `.skill` file you want from the [`dist/`](dist/) folder.
2. In Claude, open **Settings → Capabilities → Skills** and make sure Code Execution and File Creation is enabled.
3. Choose **Upload skill** and select the `.skill` file.
4. The skill loads automatically whenever a request matches its description.

### Claude Code

Copy a skill folder into your personal or project skills directory:

```bash
git clone https://github.com/<your-username>/AmelTech-Claude-Skills.git
cp -r AmelTech-Claude-Skills/skills/ameltech-class ~/.claude/skills/
```

Use `.claude/skills/` inside a project instead of `~/.claude/skills/` to share a skill with that project only.

### Claude API

Upload a skill folder with the Skills API and reference it by its `skill_id` in the `container` parameter of your Messages request. See the [Agent Skills documentation](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview).

## Usage examples

| Skill | Try saying |
|---|---|
| AmelTech Class | "Solve this question paper step by step and give me the PDF and LaTeX." · "Find the mistake in this derivation." · "Draw the circuit and plot the graph for this lab experiment." |
| AmelTech Chat Efficiency | "Efficient mode: just the answer." · "Run the council on this decision." · "How many tokens did that use?" |

## Requirements

The skills run in Claude's code-execution sandbox, which already provides most tools. For local use (e.g., Claude Code) you may need:

| Skill | Dependencies |
|---|---|
| AmelTech Class | Python 3, numpy, sympy, scipy, Pillow, pdfplumber; TeX Live (pdflatex / XeLaTeX) |
| AmelTech Chat Efficiency | Python 3 (standard library only) |

## Notes

- Skills that mention `ameltech-pdf-editor` or `ameltech-file` only use those names to route requests elsewhere; each skill here works on its own.
- AmelTech Chat Efficiency reports token savings as MEASURED, ESTIMATED or UNAVAILABLE. Savings are a target, not a guarantee.
- Always review generated papers, solutions and calculations before submitting or publishing them.

## Contributing

Issues and pull requests are welcome. When changing a skill, keep the `name` in the `SKILL.md` frontmatter identical to its folder name and keep the `description` under 1024 characters.

## License

Released under the [MIT License](LICENSE). © 2026 AmelTech.
