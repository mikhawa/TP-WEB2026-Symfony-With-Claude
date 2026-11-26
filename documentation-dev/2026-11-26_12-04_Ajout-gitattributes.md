# Ajout du fichier .gitattributes

- **Date :** 2026-11-26 12:04
- **Agent :** session principale <Modele: opus>

## Fichier(s) modifié(s)

- `.gitattributes` (création)

## Résumé des changements

- Normalisation des fins de ligne en LF (`* text=auto eol=lf`).
- Exceptions CRLF pour les scripts Windows (`.bat`, `.cmd`, `.ps1`).
- Fichiers binaires (images, polices, PDF, archives) marqués `binary` : aucune conversion.

## Raison

17 fichiers apparaissaient comme modifiés alors que seules leurs fins de ligne avaient changé (LF → CRLF). Ce `.gitattributes` évite ces diffs parasites et protège les scripts shell exécutés dans les conteneurs Docker.
