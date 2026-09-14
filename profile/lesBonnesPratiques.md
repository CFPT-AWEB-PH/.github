# Bonnes pratiques de programmation

Ce document rassemble les principes, méthodes et acronymes les plus utiles pour écrire des programmes **lisibles, fiables, testables, sécurisés et maintenables**. Il est conçu comme un support de cours ou une fiche de référence pratique, avec des exemples orientés Python, projets Web et systèmes embarqués.

> Idée directrice : écrire du code qui soit facile à comprendre, difficile à mal utiliser, simple à vérifier et raisonnablement facile à faire évoluer.

---

## 1. Les fondations d’un code de qualité

Un programme de qualité ne se limite pas à « fonctionner aujourd’hui ». Il doit aussi permettre à une autre personne — ou à son auteur plusieurs mois plus tard — de comprendre rapidement son rôle, corriger un défaut et ajouter une fonctionnalité sans provoquer de régression.

Les objectifs principaux sont :

- **Lisibilité** : le code communique clairement son intention.
- **Simplicité** : la solution ne contient pas plus de complexité que nécessaire.
- **Fiabilité** : les cas normaux, erreurs et valeurs limites sont pris en compte.
- **Testabilité** : les parties importantes peuvent être vérifiées automatiquement.
- **Maintenabilité** : les changements sont localisés et peu risqués.
- **Sécurité** : les données externes et secrets sont traités avec prudence.
- **Traçabilité** : les modifications sont suivies, revues et reproductibles.

---

## 2. Lire et structurer le code

### 2.1 Choisir des noms explicites

Un bon nom réduit le besoin de commentaire. Il décrit le rôle ou l’intention, et non un détail d’implémentation temporaire.

```python
# À éviter
def c(p, t):
    return p * (1 + t)

# Préférable
def calculer_prix_ttc(prix_ht: float, taux_tva: float) -> float:
    return prix_ht * (1 + taux_tva)
```

Bonnes habitudes :

- Nommer les variables selon leur sens : `temperature_celsius`, `nombre_absences`, `utilisateur_connecte`.
- Nommer les fonctions avec une action : `charger_configuration`, `envoyer_alerte`, `calculer_moyenne`.
- Nommer les booléens comme une question : `est_valide`, `a_droits_admin`, `connexion_active`.
- Éviter les abréviations opaques telles que `x`, `tmp`, `data` ou `cfg` quand elles cachent le rôle réel de la donnée.
- Respecter les conventions du langage et du projet : en Python, les fonctions et variables sont généralement en `snake_case` ; en JavaScript, elles sont souvent en `camelCase`.

### 2.2 Garder les fonctions ciblées

Une fonction devrait avoir un objectif principal, un comportement prévisible et une taille raisonnable. Si une fonction lit un fichier, calcule une valeur, affiche un résultat et envoie un e-mail, elle a probablement trop de responsabilités.

```python
from dataclasses import dataclass

@dataclass
class Mesure:
    valeur: float
    unite: str


def convertir_celsius_en_fahrenheit(temperature: Mesure) -> Mesure:
    if temperature.unite != "C":
        raise ValueError("La température doit être exprimée en degrés Celsius.")

    return Mesure(
        valeur=temperature.valeur * 9 / 5 + 32,
        unite="F",
    )
```

Conseils :

- Limiter le nombre de paramètres ; au-delà de trois ou quatre paramètres liés, utiliser éventuellement une `dataclass` ou un objet de configuration.
- Retourner une valeur explicite plutôt que modifier silencieusement des données globales.
- Séparer la logique métier des entrées/sorties : réseau, fichiers, écran, base de données, matériel.
- Donner à chaque fonction un contrat clair : entrées attendues, résultat, erreurs possibles.

### 2.3 Commenter le pourquoi

Le code devrait expliquer le **quoi**. Les commentaires expliquent surtout le **pourquoi** : contrainte matérielle, choix métier, limite d’une bibliothèque, compromis de performance ou comportement non évident.

```python
# Le capteur produit parfois des valeurs incohérentes juste après l’alimentation.
# Les trois premières mesures sont donc ignorées.
mesures_utiles = mesures[3:]
```

Utiliser des docstrings pour les modules, classes et fonctions publiques. Un fichier `README.md` devrait présenter au minimum le but du projet, l’installation, l’exécution, la configuration, les dépendances et les tests.

---

## 3. Prévenir les erreurs et sécuriser le code

### 3.1 Valider les entrées

Toute donnée provenant d’un utilisateur, d’un fichier, d’un capteur, d’un réseau ou d’une API doit être considérée comme potentiellement invalide.

```python
def definir_luminosite(pourcentage: int) -> int:
    if not isinstance(pourcentage, int):
        raise TypeError("La luminosité doit être un entier.")

    if not 0 <= pourcentage <= 100:
        raise ValueError("La luminosité doit être comprise entre 0 et 100.")

    return pourcentage
```

À vérifier selon le contexte :

- Type de donnée.
- Présence d’une valeur.
- Plage autorisée.
- Longueur maximale.
- Format : e-mail, date, identifiant, adresse IP, JSON, etc.
- Droits de la personne ou du système qui envoie la demande.

### 3.2 Gérer les erreurs explicitement

Éviter `except:` sans type d’exception : cette forme peut masquer une erreur de programmation ou un problème système. Intercepter les erreurs attendues, ajouter du contexte, puis laisser les erreurs inattendues remonter ou les journaliser correctement.

```python
from pathlib import Path
import json


def charger_configuration(chemin: Path) -> dict:
    try:
        return json.loads(chemin.read_text(encoding="utf-8"))
    except FileNotFoundError as erreur:
        raise RuntimeError(f"Fichier de configuration absent : {chemin}") from erreur
    except json.JSONDecodeError as erreur:
        raise RuntimeError(f"Configuration JSON invalide : {chemin}") from erreur
```

### 3.3 Sécurité de base

- Ne jamais placer de mot de passe, jeton d’API, clé privée ou secret dans le code source ni dans Git.
- Utiliser des variables d’environnement, un gestionnaire de secrets ou un fichier `.env` exclu par `.gitignore`.
- Valider toutes les entrées externes côté serveur, même si l’interface utilisateur les valide déjà.
- Utiliser des requêtes paramétrées ou un ORM pour éviter l’injection SQL.
- Vérifier les autorisations avant une opération sensible.
- Mettre à jour les bibliothèques et examiner leurs vulnérabilités connues.
- Relire et tester tout code généré par IA, en particulier les parties liées aux accès, au réseau, aux requêtes SQL et à l’authentification.
- Appliquer le principe du moindre privilège : un service ne reçoit que les droits nécessaires à son rôle.

---

## 4. Tester, formater et automatiser

### 4.1 Les tests

Les tests automatisés servent principalement à détecter les régressions. Une modification qui casse un comportement déjà validé est détectée rapidement.

Il est utile de tester :

- Les cas normaux.
- Les valeurs limites : zéro, minimum, maximum, valeur juste avant/après un seuil.
- Les entrées invalides.
- Les règles métier importantes.
- Les comportements en cas de panne d’un capteur, d’un fichier ou du réseau.

```python
import pytest

from mon_projet.prix import calculer_prix_ttc


def test_calculer_prix_ttc_avec_tva():
    assert calculer_prix_ttc(100, 0.081) == pytest.approx(108.10)


def test_calculer_prix_ttc_accepte_un_prix_nul():
    assert calculer_prix_ttc(0, 0.081) == 0
```

La couverture de code peut aider à identifier des zones non testées, mais un pourcentage élevé ne garantit pas la pertinence des tests. Il vaut mieux quelques tests solides sur les règles critiques que de nombreux tests superficiels.

### 4.2 Outils de qualité en Python

Une chaîne simple et efficace pour un projet Python moderne :

- `ruff format` : formatage automatique.
- `ruff check` : analyse statique et règles de style.
- `pytest` : tests automatisés.
- `mypy` ou `pyright` : vérification optionnelle des types.
- `pre-commit` : exécution automatique des contrôles avant un commit Git.
- GitHub Actions ou GitLab CI : exécution des contrôles dans une intégration continue.

Exemple de routine locale :

```bash
ruff format .
ruff check .
pytest
```

### 4.3 Git et travail collaboratif

- Faire des commits petits, cohérents et faciles à relire.
- Donner des messages de commit utiles : `feat: ajouter la lecture du BME280`, `fix: gérer l’absence de carte SD`, `docs: préciser l’installation`.
- Relire le diff avant de valider un commit.
- Créer une branche par fonctionnalité ou correction.
- Utiliser une Pull Request pour faire vérifier le code, discuter les choix et lancer la CI avant fusion.
- Ne pas versionner les fichiers générés, secrets, environnements virtuels ou caches.

---

## 5. Glossaire des acronymes

### 5.1 Principes de conception

| Acronyme | Signification | Explication | Projet minimal ou exemple |
|---|---|---|---|
| **KISS** | *Keep It Simple, Stupid* | Choisir la solution la plus simple qui répond correctement au besoin. « Simple » ne signifie pas négligé : la solution doit rester claire et fiable. | **Feu de température** : lire une température, choisir vert sous 25 °C et rouge au-dessus, piloter une LED. Trois fonctions suffisent : `lire_temperature()`, `choisir_couleur()`, `allumer_led()`. |
| **DRY** | *Don’t Repeat Yourself* | Une règle, une information ou une décision métier doit avoir une source unique. Il s’agit surtout d’éviter de dupliquer une même connaissance. | **Calcul de TVA** : définir `TAUX_TVA = 0.081` une seule fois et appeler `calculer_prix_ttc(prix_ht)`. |
| **YAGNI** | *You Aren’t Gonna Need It* | Ne pas développer une fonctionnalité pour un besoin hypothétique. Ajouter une fonction lorsqu’un besoin réel, actuel et vérifié apparaît. | **Liste de tâches** : commencer avec ajout, affichage et suppression dans un fichier JSON. Ne pas créer immédiatement comptes utilisateurs, synchronisation cloud ou export PDF. |
| **SoC** | *Separation of Concerns* | Séparer les préoccupations : interface, logique métier, stockage, réseau, matériel, etc. | **Station météo** : `capteur.py` lit le BME280, `metier.py` interprète les seuils, `affichage.py` pilote l’OLED et `main.py` orchestre. |
| **SRP** | *Single Responsibility Principle* | Une fonction, classe ou module doit avoir une responsabilité principale et donc une raison principale de changer. | **Bulletins scolaires** : `charger_eleves()` lit un CSV, `calculer_moyenne()` calcule la moyenne, `generer_bulletin()` produit le texte. |
| **OCP** | *Open/Closed Principle* | Un composant peut être étendu sans devoir modifier sa logique centrale à chaque ajout. | **Export de mesures** : partir d’une interface `exporter(mesures)` et ajouter `ExportCSV`, `ExportJSON`, puis `ExportHTML` sans une longue chaîne de conditions. |
| **LSP** | *Liskov Substitution Principle* | Une instance d’une sous-classe doit pouvoir remplacer une instance de sa classe de base sans casser le comportement attendu. | **Alerte** : le programme reçoit un objet avec `activer()` et `desactiver()`. Une LED, un buzzer ou une notification doivent pouvoir être substitués sans surprise. |
| **ISP** | *Interface Segregation Principle* | Préférer plusieurs interfaces petites et adaptées plutôt qu’une interface trop large qui force des méthodes inutiles. | **Périphériques** : `Imprimable` fournit `imprimer()` ; `Scannable` fournit `scanner()`. Une imprimante simple n’a pas à implémenter une fausse méthode `scanner()`. |
| **DIP** | *Dependency Inversion Principle* | La logique métier dépend d’abstractions, et non d’implémentations techniques précises. | **Stockage de mesures** : `StationMeteo` dépend d’un contrat `Stockage.sauvegarder()`. On branche une version JSON, SQLite ou en mémoire pour les tests. |
| **CQS** | *Command–Query Separation* | Une opération devrait soit modifier l’état, soit renvoyer une information, mais éviter de faire les deux. | **Tirelire** : `solde()` lit et retourne le solde ; `deposer(10)` le modifie. La lecture ne modifie jamais la tirelire. |
| **POLA** | *Principle of Least Astonishment* | Une fonction ou une API se comporte de la manière la moins surprenante possible pour son utilisateur. | **Copie de configuration** : `copier_configuration()` crée une copie ; elle ne doit ni supprimer l’original, ni redémarrer un service sans le dire. |
| **PIE** | *Program Intently and Expressively* | Exprimer l’intention métier par les noms et structures plutôt que montrer seulement des opérations bas niveau. | **Arrosage** : écrire `doit_arroser(humidite_sol, seuil)` plutôt que disperser partout `if humidite < 38`. |
| **WET** | *Write Everything Twice* ou *We Enjoy Typing* | Terme humoristique opposé à DRY : il désigne une duplication excessive de code ou de règles. | **Anti-exemple** : copier le même test de seuil de température dans l’écran OLED, le contrôle des LEDs et l’alerte réseau. Corriger avec `etat_thermique(temperature)`. |

### 5.2 Méthodes et qualité logicielle

| Acronyme | Signification | Explication | Projet minimal ou exemple |
|---|---|---|---|
| **TDD** | *Test-Driven Development* | Écrire d’abord un test qui échoue, implémenter le minimum pour le faire réussir, puis améliorer le code. Cycle : rouge, vert, refactorisation. | **Conversion de température** : écrire `assert celsius_vers_fahrenheit(0) == 32`, implémenter la fonction, puis ajouter 100 °C et −40 °C. |
| **BDD** | *Behavior-Driven Development* | Décrire le comportement attendu dans un langage proche du besoin utilisateur, souvent « Étant donné / Quand / Alors ». | **Serrure à code** : « Étant donné une serrure verrouillée, quand le bon code est saisi, alors elle se déverrouille. » Le scénario devient un test. |
| **CI** | *Continuous Integration* | Intégrer fréquemment les changements et exécuter automatiquement tests, lint, formatage, compilation et autres contrôles. | **Calculatrice GitHub** : à chaque `git push`, GitHub Actions exécute `ruff check .` et `pytest`. |
| **CD** | *Continuous Delivery* ou *Continuous Deployment* | *Delivery* produit automatiquement une version déployable ; *Deployment* la déploie automatiquement après les validations prévues. | **Application Flask** : si les tests passent, la CI construit une image Docker. En livraison continue elle est prête à publier ; en déploiement continu elle est publiée automatiquement. |
| **VCS** | *Version Control System* | Système de gestion de versions tel que Git : historique, comparaison, restauration, travail en branche et fusion. | **Projet CircuitPython** : versionner `code.py`, `README.md` et la configuration non sensible ; faire un commit après chaque étape fonctionnelle. |
| **PR** | *Pull Request* | Proposition de fusion de changements vers une branche cible, accompagnée d’une revue, de discussions et souvent de contrôles automatiques. | **Projet à deux personnes** : créer `feature-affichage-oled`, pousser les changements, ouvrir une PR vers `main`, relire avant fusion. |
| **IDE** | *Integrated Development Environment* | Environnement regroupant éditeur, terminal, débogueur, outils de test, gestion de projet et souvent Git. | **Tri d’images en Python** : utiliser VS Code ou PyCharm, poser un point d’arrêt dans la fonction de classement et inspecter les variables. |

### 5.3 Architecture, données et Web

| Acronyme | Signification | Explication | Projet minimal ou exemple |
|---|---|---|---|
| **MVC** | *Model–View–Controller* | Sépare les données et règles métier (*Model*), l’affichage (*View*) et le traitement des actions (*Controller*). | **Compteur de clics** : le modèle stocke le nombre, la vue l’affiche, le contrôleur traite le bouton « +1 ». |
| **DTO** | *Data Transfer Object* | Objet simple qui transporte des données entre couches ou services, sans porter toute la logique métier. | **Inscription à un atelier** : `InscriptionDTO(nom, email, atelier)` transporte les données du formulaire vers le service d’inscription. |
| **ORM** | *Object-Relational Mapping* | Outil qui relie des objets du programme aux tables d’une base relationnelle. | **Carnet d’adresses Flask** : avec SQLAlchemy, une classe `Contact` représente une ligne de la table `contacts`. |
| **CRUD** | *Create, Read, Update, Delete* | Les quatre opérations de base sur des données persistantes : créer, lire, modifier et supprimer. | **Répertoire de contacts** : créer un contact, consulter la liste, modifier un numéro, supprimer un contact. |
| **ACID** | *Atomicity, Consistency, Isolation, Durability* | Propriétés recherchées pour les transactions de base de données : tout ou rien, cohérence, isolation des opérations concurrentes et persistance après validation. | **Virement fictif** : retirer 20 CHF du compte A et ajouter 20 CHF au compte B dans une transaction unique ; en cas d’erreur, l’opération entière est annulée. |
| **API** | *Application Programming Interface* | Contrat permettant à deux logiciels ou composants de communiquer par des fonctions, messages ou requêtes. | **Module de conversions** : exposer `celsius_vers_fahrenheit()` et `km_vers_miles()` dans `conversions.py` ; ces fonctions sont l’API publique du module. |
| **REST** | *Representational State Transfer* | Style d’API Web utilisant des ressources identifiées par URL et les méthodes HTTP usuelles. Les représentations sont souvent en JSON. | **API de tâches Flask** : `GET /taches`, `POST /taches`, `PATCH /taches/3`, `DELETE /taches/3`. |
| **HTTP** | *Hypertext Transfer Protocol* | Protocole de requête-réponse utilisé par le Web entre un client et un serveur. | **Route Flask** : `GET /bonjour` retourne `{"message": "Bonjour"}` avec un code HTTP et des en-têtes. |
| **JSON** | *JavaScript Object Notation* | Format texte structuré, léger et courant pour fichiers de configuration et échanges Web. | **Tâches locales** : stocker `[{"titre": "Préparer le cours", "faite": false}]` dans `taches.json`. |

### 5.4 Sécurité et authentification

| Acronyme | Signification | Explication | Projet minimal ou exemple |
|---|---|---|---|
| **JWT** | *JSON Web Token* | Jeton signé, souvent utilisé par une API pour porter une preuve d’authentification ou des informations d’autorisation. Son contenu peut généralement être décodé : il ne faut donc jamais y placer de secret. | **API Flask protégée** : `POST /connexion` renvoie un jeton après vérification ; `GET /profil` exige ensuite un jeton valide dans l’en-tête `Authorization`. |
| **OWASP** | *Open Worldwide Application Security Project* | Communauté et ressources de référence sur la sécurité applicative, notamment pour identifier les risques Web fréquents. | **Connexion sécurisée** : mots de passe hachés, validation des entrées, requêtes SQL paramétrées, limitation des essais et secrets absents de Git. |

---

## 6. Exemple fil rouge : API de tâches pour une classe

Un petit gestionnaire de tâches est un bon projet pour exercer progressivement la majorité des notions du glossaire.

### 6.1 Version 1 : le minimum fonctionnel

Créer une application Python qui gère des tâches :

- Une tâche contient un identifiant, un titre, un état `faite` et éventuellement une échéance.
- Les données sont enregistrées dans `taches.json`.
- Le programme peut ajouter, lister, modifier et supprimer une tâche.

Cette première version exerce : **KISS**, **YAGNI**, **JSON** et **CRUD**.

### 6.2 Version 2 : organisation claire

Proposer une structure simple :

```text
projet_taches/
├── app.py             # Point d’entrée ou routes Flask
├── metier.py          # Règles : validation, création, changement d’état
├── stockage.py        # Lecture/écriture JSON ou SQLite
├── modeles.py         # Dataclasses : Tache, éventuellement DTO
├── tests/
│   ├── test_metier.py
│   └── test_api.py
├── pyproject.toml
├── README.md
└── .gitignore
```

Cette étape exerce : **SoC**, **SRP**, **PIE** et **VCS**.

### 6.3 Version 3 : tests et automatisation

- Écrire les tests des règles métier avant ou pendant l’implémentation : **TDD**.
- Écrire des scénarios utilisateur : **BDD**.
- Ajouter `ruff`, `pytest`, et éventuellement `pyright`.
- Configurer GitHub Actions pour lancer les contrôles à chaque push : **CI**.
- Travailler dans des branches et utiliser les Pull Requests : **PR**.

### 6.4 Version 4 : API Web

Exposer les tâches via Flask :

```text
GET    /taches        -> liste les tâches
POST   /taches        -> crée une tâche
GET    /taches/<id>   -> retourne une tâche
PATCH  /taches/<id>   -> modifie une tâche
DELETE /taches/<id>   -> supprime une tâche
```

Cette étape exerce : **API**, **REST**, **HTTP**, **JSON** et **MVC** ou une séparation comparable.

### 6.5 Version 5 : extension raisonnable

- Remplacer le stockage JSON par SQLite sans modifier les règles métier : **DIP**.
- Ajouter une authentification JWT seulement si l’application devient privée : **YAGNI**.
- Déployer automatiquement une version validée, lorsque l’usage le justifie : **CD**.
- Consulter les recommandations OWASP avant de rendre le service accessible publiquement : **OWASP**.

---

## 7. Checklist avant livraison

### Lisibilité

- Les noms des variables, fonctions et classes sont-ils explicites ?
- Chaque fonction possède-t-elle un rôle clair ?
- Les commentaires expliquent-ils une intention ou une contrainte, plutôt que répéter le code ?
- Le projet contient-il un README compréhensible ?

### Fiabilité

- Les entrées externes sont-elles validées ?
- Les cas d’erreur attendus sont-ils traités ?
- Les valeurs limites sont-elles couvertes ?
- Les messages d’erreur donnent-ils un contexte exploitable ?

### Tests et qualité

- Les règles métier importantes disposent-elles de tests ?
- Les tests réussissent-ils avant livraison ?
- Le formateur et le linter ont-ils été exécutés ?
- Les modifications ont-elles été relues dans le diff Git ?

### Sécurité

- Aucun secret n’est-il présent dans le dépôt ?
- Les dépendances sont-elles à jour ?
- Les requêtes SQL sont-elles paramétrées ou gérées par un ORM ?
- Les autorisations sont-elles vérifiées côté serveur ?
- Les données sensibles sont-elles absentes des logs, des JWT et des messages d’erreur ?

### Évolution

- La solution reste-t-elle proportionnée au besoin réel ?
- Les règles répétées sont-elles réellement centralisées ?
- Les composants techniques peuvent-ils être remplacés sans réécrire toute la logique métier ?
- A-t-on évité d’ajouter des fonctionnalités non demandées ?

---

## 8. Ressources recommandées

### Documentation et styles de code

- [PEP 8 — Guide de style Python](https://peps.python.org/pep-0008/)
- [PEP 257 — Conventions pour les docstrings Python](https://peps.python.org/pep-0257/)
- [Documentation Python](https://docs.python.org/fr/3/)
- [Ruff — linter et formateur Python](https://docs.astral.sh/ruff/)
- [pytest — documentation](https://docs.pytest.org/)
- [Pyright — vérification de types Python](https://microsoft.github.io/pyright/)
- [pre-commit — hooks Git](https://pre-commit.com/)

### Git, collaboration et automatisation

- [Documentation officielle Git](https://git-scm.com/doc)
- [GitHub Skills — exercices pratiques GitHub](https://skills.github.com/)
- [Documentation GitHub Actions](https://docs.github.com/actions)
- [GitLab CI/CD](https://docs.gitlab.com/ci/)

### Conception et architecture

- [The Twelve-Factor App](https://12factor.net/) — bonnes pratiques pour applications Web et services
- [Refactoring.Guru — SOLID](https://refactoring.guru/design-patterns/solid)
- [Refactoring.Guru — Design Patterns](https://refactoring.guru/design-patterns)
- [Martin Fowler — articles sur l’architecture et le refactoring](https://martinfowler.com/)

### APIs et applications Web

- [Documentation Flask](https://flask.palletsprojects.com/)
- [MDN Web Docs — HTTP](https://developer.mozilla.org/fr/docs/Web/HTTP)
- [MDN Web Docs — JSON](https://developer.mozilla.org/fr/docs/Learn_web_development/Core/Scripting/JSON)
- [MDN Web Docs — API Web](https://developer.mozilla.org/fr/docs/Learn_web_development/Extensions/Client-side_APIs/Introduction)
- [SQLAlchemy — documentation](https://docs.sqlalchemy.org/)

### Sécurité

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Python Security — bonnes pratiques](https://python.readthedocs.io/en/latest/library/security_warnings.html)

### Systèmes embarqués et Raspberry Pi

- [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)
- [CircuitPython Documentation](https://docs.circuitpython.org/)
- [Adafruit Learning System](https://learn.adafruit.com/)
- [Adafruit Blinka](https://learn.adafruit.com/circuitpython-on-raspberrypi-linux)

---

## Conclusion

Les bonnes pratiques ne sont pas une liste de règles rigides à appliquer mécaniquement. Elles servent à prendre de meilleures décisions : conserver la simplicité, isoler les responsabilités, automatiser les contrôles, protéger les données et ne construire que ce dont le projet a réellement besoin.

Pour progresser efficacement : commencez par **KISS**, **DRY**, des noms explicites, des tests ciblés, Git et un formatage automatique. Ajoutez ensuite les principes SOLID, la CI, les abstractions et les mécanismes de sécurité lorsque la taille ou les contraintes du projet les rendent réellement utiles.
