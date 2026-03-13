# Team AI Skills — Gemini CLI & Claude Code

Skills partagés pour Gemini CLI et Claude Code, utilisables sur tous les projets Android de l'équipe.

## Installation (une seule fois par développeur)

### Gemini CLI — installation globale
```bash
# Installer un skill depuis GitHub
gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/code-review
gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/document

# Ou depuis un chemin local (pendant le développement)
gemini skills link /chemin/vers/iaSkills/skills/code-review
gemini skills link /chemin/vers/iaSkills/skills/document
```

### Claude Code — installation globale
```bash
# Cloner le repo puis copier les skills Claude
git clone https://github.com/HAGUI-85/iaSkills.git
cp -r iaSkills/skills/code-review/claude ~/.claude/skills/code-review
cp -r iaSkills/skills/document/claude ~/.claude/skills/document
```

### Vérifier l'installation
```bash
# Gemini
gemini skills list --all | grep -E "code-review|document"

# Claude Code — taper dans le chat :
/code-review .
/document .
```

---

## Skills disponibles

| Skill | Commande | Outils | Description |
|-------|----------|--------|-------------|
| Code Review | `/code-review [scope]` | Gemini CLI + Claude Code | Revue complète : archi, sécu, exceptions, patterns, perf, tests |
| Document | `/document [scope]` | Gemini CLI + Claude Code | Génère une `documentation.md` complète : contexte, use cases, diagrammes de séquence, détails de code, guide de setup |

---

## Structure du repo

```
iaSkills/
├── README.md
└── skills/
    ├── code-review/
    │   ├── SKILL.md          ← Gemini CLI version
    │   └── claude/
    │       └── SKILL.md      ← Claude Code version
    └── document/
        ├── SKILL.md          ← Gemini CLI version
        └── claude/
            └── SKILL.md      ← Claude Code version
```

---

## Usage

### Gemini CLI
```bash
# Code Review
gemini /code-review .
gemini /code-review core/logger
gemini /code-review app/src/main/java/com/example/utils/TokenGenerator.kt
gemini --yolo -p "/code-review ."   # mode headless (CI)

# Documentation
gemini /document .
gemini /document core/feature-login
```

### Claude Code
```bash
# Dans le chat Claude Code :
/code-review .
/code-review core/logger
/document .
/document core/feature-login
```

---

## Ajouter un skill à ce repo

1. Créer `skills/<nom-du-skill>/SKILL.md` (Gemini CLI)
2. Créer `skills/<nom-du-skill>/claude/SKILL.md` (Claude Code)
3. Tester localement :
   ```bash
   gemini skills link ./skills/<nom-du-skill>
   cp -r skills/<nom-du-skill>/claude ~/.claude/skills/<nom-du-skill>
   ```
4. Ouvrir une PR — après merge, les devs mettent à jour :
   ```bash
   gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/<nom-du-skill>
   ```

---

## Mise à jour d'un skill installé

```bash
# Gemini CLI
gemini skills uninstall code-review && gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/code-review
gemini skills uninstall document    && gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/document

# Claude Code
git clone https://github.com/HAGUI-85/iaSkills.git /tmp/iaSkills
cp -r /tmp/iaSkills/skills/code-review/claude ~/.claude/skills/code-review
cp -r /tmp/iaSkills/skills/document/claude    ~/.claude/skills/document
```
