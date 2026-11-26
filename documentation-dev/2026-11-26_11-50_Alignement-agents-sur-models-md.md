# Alignement des agents sur `.claude/models.md`

**Date :** 2026-11-26 11:50
**Branche :** `feature/01-pre-install`

Agent: claude-principal <Modele: opus>

## Fichiers modifiés

- `.claude/agents/*.md` (9 agents, tous sauf `verificateur`) : ajout d'une section « Fin de mission » (ni commit ni branche, compte rendu fichiers / résumé / raison reporté par la session principale dans `documentation-dev/`).
- `.claude/agents/migrations.md` : rôle étendu à la réparation de migrations et à la cohérence schéma ↔ entités ; la maintenance des factories Foundry est retirée (passe à `testeur`).
- `.claude/agents/testeur.md` : prend en charge les factories (`src/Factory/`) et stories (`src/Story/`) Foundry et les tests de non-régression.
- `.claude/agents/devops.md` : ajout du dossier `docker/` et du déploiement VPS de préproduction (après `auditeur-securite`).
- `.claude/agents/debogueur.md` : cas d'usage élargis (comportement intermittent, échec non résolu par `dev-symfony`) ; le test de non-régression est délégué à `testeur`.
- `.claude/agents/dev-symfony.md` : consigne de fin fusionnée dans « Fin de mission ».
- `CLAUDE.md` §6 : renvoi aux enchaînements types de `models.md` et règle « en cas de doute, prendre le niveau le plus élevé ».

## Raison

`.claude/models.md` a été réécrit (répartition des rôles, enchaînements types, compte rendu des agents) : les définitions d'agents et `CLAUDE.md` devaient refléter cette nouvelle répartition.
