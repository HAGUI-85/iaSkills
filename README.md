# Team AI Skills — Gemini CLI & Claude Code

Skills partagés pour Gemini CLI et Claude Code, utilisables sur tous les projets Android de l'équipe.

## Installation (une seule fois par développeur)

### Gemini CLI — installation globale
```bash
# Depuis GitHub
gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/code-review

# Ou depuis un chemin local (pendant le développement)
gemini skills link /chemin/vers/iaSkills/skills/code-review
```

### Claude Code — installation globale
```bash
# Cloner le repo puis copier le skill Claude
git clone https://github.com/HAGUI-85/iaSkills.git
cp -r iaSkills/skills/code-review/claude ~/.claude/skills/code-review
```

### Vérifier l'installation
```bash
# Gemini
gemini skills list --all | grep code-review

# Claude Code — taper dans le chat :
/code-review .
```

---

## Skills disponibles

| Skill | Commande | Outils | Description |
|-------|----------|--------|-------------|
| Code Review | `/code-review [scope]` | Gemini CLI + Claude Code | Revue complète : archi, sécu, exceptions, patterns, perf, tests |

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
    └── (futurs skills ici)
```

---

## Usage

### Gemini CLI
```bash
# Review du projet entier
gemini /code-review .

# Review d'un module
gemini /code-review core/logger

# Review d'un fichier
gemini /code-review app/src/main/java/com/chronopost/hub/utils/TokenGenerator.kt

# Mode headless (CI / non-interactif)
gemini --yolo -p "/code-review ."
```

### Claude Code
```bash
# Dans le chat Claude Code :
/code-review .
/code-review core/logger
/code-review app/src/main/java/com/chr/h/utils/TokenGenerator.kt
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
gemini skills uninstall code-review
gemini skills install https://github.com/HAGUI-85/iaSkills.git --path skills/code-review

# Claude Code
rm -rf ~/.claude/skills/code-review
git clone https://github.com/HAGUI-85/iaSkills.git /tmp/iaSkills
cp -r /tmp/iaSkills/skills/code-review/claude ~/.claude/skills/code-review
```
