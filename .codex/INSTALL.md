# Installing jackin-dev for Codex

Codex does not have a native plugin system. To use these skills:

1. Clone this repo alongside your project:
   ```sh
   git clone https://github.com/donbeave/jackin-dev.git
   ```

2. Reference the skills in your project's `AGENTS.md`:
   ```markdown
   ## Release Skills
   Release skills are available in `../jackin-dev/skills/`. When asked to run
   release-check, release-notes, or release, read and follow the corresponding
   SKILL.md in that directory.
   ```

3. Alternatively, symlink the skills into your project:
   ```sh
   mkdir -p .codex/skills
   ln -s /path/to/jackin-dev/skills/release-check .codex/skills/release-check
   ln -s /path/to/jackin-dev/skills/release-notes .codex/skills/release-notes
   ln -s /path/to/jackin-dev/skills/release .codex/skills/release
   ```
