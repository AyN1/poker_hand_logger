# Schéma JSON — Hand Logger (version 1)

Format maître des mains loggées. Pensé pour être relu par d'autres projets :
tout ce qui n'est pas indispensable est optionnel, et rien ne dépend de l'interface.

## Fichier exporté

```json
{
  "version": 1,
  "exported_at": "2026-10-07T21:30:00.000Z",
  "sessions": [ /* Session */ ]
}
```

L'import accepte aussi une liste de sessions seule, ou une session seule.
Fusion à l'import : par `id` de session, puis par `id` de main (la main importée remplace celle qui a le même `id`).

## Session

| Champ | Type | Notes |
|---|---|---|
| `id` | string | unique |
| `game` | string | `"NLHE"` pour l'instant (prévu pour `"PLO"` etc.) |
| `format` | `"cash"` \| `"tournament"` | |
| `venue` | string | lieu, peut être vide |
| `started_at` / `ended_at` | ISO 8601 \| null | `ended_at: null` = session en cours |
| `table_size` | int 2–10 | valeur courante de l'ancre |
| `blinds` | Blinds | valeur courante de l'ancre |
| `last_stack` | number \| null | sert à pré-remplir la main suivante |
| `hands` | Hand[] | triées par `n` |

## Blinds

`{ "sb": 1, "bb": 2, "ante": 0, "ante_mode": "bb" }`

- `ante_mode: "bb"` : big blind ante, payée une fois par la BB.
- `ante_mode: "each"` : chaque joueur paie `ante`.

## Hand

| Champ | Type | Notes |
|---|---|---|
| `id` | string | unique globalement (permet de fusionner des fichiers) |
| `n` | int | numéro dans la session |
| `time` | ISO 8601 | début de saisie |
| `flagged` | bool | main posée en marque-page, à compléter |
| `table_size` | int | **copie** de l'ancre au moment de la main |
| `blinds` | Blinds | **copie** de l'ancre au moment de la main |
| `hero` | `{pos, cards, stack}` | `stack` = stack au début de la main |
| `villains` | `[{pos, cards?}]` \| null | joueurs dans le coup ; `null` = pas encore saisi |
| `board` | `{flop, turn, river}` | `flop`: 3 cartes ou null ; `turn`/`river`: 1 carte ou null |
| `actions` | Action[] | séquence ordonnée |
| `result` | `{outcome, net, showdown}` \| null | null pour une main flaggée sans résultat |
| `tags` | string[] | `value`, `bluff`, `hero-call`, `hero-fold`, `cooler`, `mistake`, `review` |
| `note` | string | |

## Action

`{ "st": "flop", "pos": "CO", "a": "bet", "frac": 0.333 }`

| Champ | Notes |
|---|---|
| `st` | `pre` \| `flop` \| `turn` \| `river` |
| `pos` | position du joueur qui agit |
| `a` | `fold` \| `check` \| `call` \| `bet` \| `raise` |
| `to` | optionnel — montant **total** misé par le joueur sur cette street (« raise to ») |
| `frac` | optionnel — taille en fraction du pot (bet ⅓ = 0.333). Pour un raise : fraction du pot après call |
| `x` | optionnel — multiple de la mise en cours, utilisé seulement si la mise en cours était inconnue |
| `allin` | optionnel — `true` |

Un `bet`/`raise` sans `to`, `frac` ni `x` est valide : la taille est inconnue, le pot devient inconnu à partir de là.

## Conventions

- **Cartes** : rang + couleur minuscule — `As`, `Kd`, `Th`, `2c`. Rangs `A K Q J T 9 … 2`, couleurs `s h d c`.
- **Montants** : en jetons, jamais en BB. Les BB se calculent avec `blinds.bb` de la main.
- **Positions** selon `table_size` :

| Joueurs | Positions (ordre preflop) |
|---|---|
| 2 | BTN (poste la SB), BB |
| 3 | BTN, SB, BB |
| 4 | CO, BTN, SB, BB |
| 5 | HJ, CO, BTN, SB, BB |
| 6 | UTG, HJ, CO, BTN, SB, BB |
| 7 | UTG, LJ, HJ, CO, BTN, SB, BB |
| 8 | UTG, UTG+1, LJ, HJ, CO, BTN, SB, BB |
| 9 | UTG, UTG+1, MP, LJ, HJ, CO, BTN, SB, BB |
| 10 | UTG, UTG+1, UTG+2, MP, LJ, HJ, CO, BTN, SB, BB |

  Postflop : premier joueur après le BTN, le BTN parle en dernier.
- Les joueurs absents de `villains` sont considérés fold preflop sans avoir mis d'argent (leurs blindes restent dans le pot).
- **Non stocké, toujours recalculé** : pot, montant des calls, ordre de parole.
- `result.net` : gain/perte net de la main pour le hero, en jetons (rake non déduit sauf si corrigé à la main).
