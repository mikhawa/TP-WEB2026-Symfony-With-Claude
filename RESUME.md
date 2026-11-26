# Résumé commandes Claude Codes

[Retour à l'accueil](README.md)

### Les commandes à connaître

| Commande | Rôle |
| --- | --- |
| `/help` | Liste des commandes disponibles |
| `/init` | Génère ou améliore le CLAUDE.md |
| `/clear` | Vide la conversation (nouvelle tâche) |
| `/compact` | Résume la conversation pour libérer du contexte |
| `/context` | Montre ce qui occupe la fenêtre de contexte |
| `/memory` | Ouvre les fichiers CLAUDE.md et l'auto memory |
| `/permissions` | Gère les règles allow / ask / deny |
| `/model` | Change de modèle (Fable, Opus, Sonnet, Haiku…) |
| `/agents` | Crée et gère les sous-agents |
| `/mcp` | Gère les serveurs MCP connectés |
| `/hooks` | Gère les hooks |
| `/plugin` | Installe ou désactive des plugins |
| `/skills` | Liste les skills disponibles |
| `/resume` | Reprend une ancienne conversation |
| `/rewind` | Revient en arrière (code et conversation) à un point précédent |
| `/usage` | Consommation de votre forfait |
| `/doctor` | Vérifie l'installation et la configuration |
| `/status` | État de la session ; liste aussi les agents, skills et commandes mal formés, les règles de permission ignorées et les hooks dont le matcher ne correspond jamais |
| `/tasks` | Tâches en arrière-plan : shells, sous-agents, tâches planifiées, réveils de `/loop` |
| `/loop` | Relance une commande ou un prompt à intervalle régulier (jusqu'à 30 jours tant que la session tourne) |
| `/code-review` | Relecture par Claude, centrée sur les bugs et le non-respect du CLAUDE.md ; chaque point est classé Important, Pre-existing ou Nit. Sans cible, relit toute la branche par rapport à la branche par défaut |
| `/simplify` | Nettoyage du code modifié : réutilisation, simplification, efficacité |

### Raccourcis utiles

- `Shift+Tab` : change de mode (Manual, acceptEdits, plan…).
- `Échap` : interrompt Claude ; deux fois `Échap` pour revenir à un message précédent.
- `@fichier` : ajoute un fichier au contexte (`@src/Controller/FactureController.php`).
- `!commande` : lance une commande shell vous-même, sa sortie entre dans le contexte.
- `Ctrl+B` : passe une tâche longue (ou un sous-agent) en arrière-plan.