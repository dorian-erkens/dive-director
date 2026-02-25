# PRD – Inspector mode dans le CLI

> **Issue source :** #001
> **Date :** 2024-01-15
> **Statut :** Draft

---

## 1. Idea – Description de l'amelioration

- **Idea title**
  Inspector mode dans le CLI

- **Context – Pourquoi maintenant ?**
  Le CLI Dive Director orchestré avec /plan-dive masque le raisonnement des agents. Les DP et développeurs n'ont pas de visibilité sur les décisions intermédiaires, contrairement à la web app qui dispose d'un panel Inspector. Le cursus DPE-PN5 met l'accent sur la traçabilité des décisions.

- **Problem statement**
  > Les utilisateurs du CLI ne peuvent pas observer le processus de décision des agents lors de la planification de plongée, limitant leur compréhension, capacité de debug et potentiel pédagogique de l'outil.

---

## 2. Goal – Objectif structure

> **Goal**
> Augmenter la transparence du processus décisionnel des agents dans le CLI pour améliorer la compréhension utilisateur et les capacités de debug.

- **Metrique cle**
  - Nom : Taux d'utilisation du mode --inspect
  - Valeur actuelle : 0% (feature inexistante)
  - Cible visee : 40% des sessions /plan-dive **[Assumption]**

- **Impact attendu** : Amélioration de la confiance utilisateur et réduction du temps de debug

---

## 3. ICE Score + Confidence Meter

- **Impact (I)** : 6 / 10
- **Ease (E)** : 8 / 10
- **Confidence (C)** : 2.0 / 10

### Evidence collectee

- [x] Self-conviction / intuition interne (+0.1)
- [x] Avis d'autres membres de l'equipe / stakeholders (+0.1)
- [ ] Trends marche / benchmarks sectoriels (+0.1)
- [x] Feedback client anecdotique (support, conversations) (+0.4)
- [x] Estimations / plans de faisabilite (+0.4)
- [ ] Survey quantitatif / market research (+1.0)
- [ ] Analytics / logs / funnel data (+1.0)
- [ ] Interviews de validation utilisateur (+2.5)
- [ ] User study avec prototype (+2.5-3.0)
- [ ] MVP / feature live + behavioral metrics (+3.0)

**Score Confidence (C) = :** `2.0`

| Score | Niveau | Action |
|-------|--------|--------|
| 0.1-0.5 | Very Low | Valider avant prototypage |
| 0.5-1.5 | Low | Discovery legere |
| 1.5-3.0 | Medium | Prototype + test interne |
| 3.0-6.0 | High | Passer en delivery |
| 6.0-10 | Very High | Priorite forte |

**ICE = (I x C x E) / 10 = 9.6**

---

## 4. Hypotheses a tester (Discovery)

### H1
- **Hypothese :**
  > Les utilisateurs CLI souhaitent comprendre le raisonnement des agents pendant la planification de plongée et utiliseront activement le mode --inspect.
- **Test :** Prototype avec mode --inspect sur 3-5 scénarios de planification typiques, test avec 5 DP/développeurs
- **Critere de validation :** 4/5 testeurs activent spontanément le mode --inspect et trouvent les informations utiles
- **Criticite :** Must validate

### H2
- **Hypothese :**
  > L'affichage temps réel des décisions d'agents améliore la confiance dans les résultats de planification.
- **Test :** A/B test : même planification avec/sans mode inspect, mesure du niveau de confiance (échelle 1-5)
- **Critere de validation :** Score confiance augmente de +1 point en moyenne avec le mode inspect
- **Criticite :** Nice to have

---

## 5. Prototype Scope

### 5.1 User flows critiques

**Flow 1 : Planification avec inspection**
- Trigger : Utilisateur lance `dive-cli /plan-dive --inspect "Plongée Épave 35m, nitrox, 2 plongeurs N2"`
- Etapes :
  1. L'utilisateur exécute la commande avec le flag --inspect
  2. Le système affiche en temps réel : "🤖 Agent: PlanningAgent", "🔧 Outil: DepthAnalyzer", "💭 Décision: Nitrox 32% recommandé pour 35m"
  3. Les informations défilent progressivement pendant l'orchestration des agents
- Outcome : Plan de plongée complet avec trace complète du raisonnement

### 5.2 Logique fonctionnelle

- Si --inspect activé, alors chaque appel d'agent/outil génère une ligne de log formatée
- Dans l'état inspection, l'utilisateur peut voir les transitions entre agents (Planning → Safety → Equipment)
- Si erreur d'agent, le mode inspect affiche le contexte de l'erreur pour faciliter le debug

### 5.3 Donnees mockees

- Types de donnees : Logs d'agents, appels d'outils, décisions intermédiaires
- Attributs : timestamp, nom_agent, outil_utilisé, décision_prise, niveau_confiance
- Exemples : "14:32:15 🤖 SafetyAgent > 🔧 DecompressionCalculator > 💭 Paliers: 3min@6m (confiance: 95%)"

---

## 6. Success criteria

### Qualitatif
- [ ] Les testeurs comprennent la fonctionnalite sans explication detaillee
- [ ] Le flow principal est complete sans blocage majeur
- [ ] Retour majoritairement positif sur la lisibilité et l'utilité des informations d'inspection

### Quantitatif
- [ ] >= 4 / 5 testeurs completent une planification avec --inspect sans aide
- [ ] Score satisfaction >= 4 / 5

### Decision post-test
- Si criteres atteints → passe en delivery
- Si criteres non atteints → adapter le format d'affichage ou simplifier les informations

---

## 7. Notes pour Claude Code

- **Fidelite** : Prototype fonctionnel minimal, testable en local
- **Priorite** : flows critiques + logique metier + donnees mockees
- **Assumptions** marquees explicitement dans le document