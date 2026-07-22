# lcv_skills

Personal skills for Claude Code.  
This repo is the source of truth across all work machines.

---

## ⚙️ Setup on a new machine

```bash
git clone <REPO-URL> ~/lcv/lcv_skills
ln -s ~/lcv/lcv_skills ~/.claude/skills
```

From that point, `git pull` is enough to keep skills up to date.

---

## 🔄 Adding or updating a skill

```bash
mkdir lcv_skills/skill-name
# ... write SKILL.md ...
git add . && git commit -m "feat: add skill-name" && git push
```

---

## 📦 Skills

| Skill | Description |
|-------|-------------|
| [jira-feature-scaffold](./jira-feature-scaffold/) | Generates `{ID}.md` and `{ID}_TODO.md` from a Jira ticket text |