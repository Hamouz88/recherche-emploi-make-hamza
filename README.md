# Présentation du projet : Veille d'offres d'emploi automatisée

## A. Objectif du projet
Automatiser la veille et la centralisation des offres d'emploi sur un rôle précis (ex: Développeur Web) dans une ville ciblée (ex: Lyon) publiées sur la semaine écoulée, afin d'optimiser le temps de recherche et d'assurer un suivi structuré des candidatures.

## B. Outils utilisés
* **Make** : Plateforme d'orchestration / d'automatisation des flux de données.
* **API JSearch (RapidAPI)** : API de recherche d'emploi agrégeant les offres de multiples plateformes (LinkedIn, Indeed, Glassdoor, etc.).
* **Google Sheets** : Base de données de suivi des candidatures.
* **Discord** : Canal d'alertes instantanées pour chaque nouvelle offre détectée.

## C. Rôle de chaque module du scénario
1. **Schedule Setting / Clock** : Déclenche le scénario automatiquement 1 fois par semaine (ex: tous les lundis à 8h00).
2. **HTTP - Make a request** : Interroge l'endpoint `/search` de l'API JSearch via la méthode GET avec les paramètres de recherche (`query`, `country`, `date_posted`) et la clé d'API passée dans les headers (`x-api-key`).
3. **Iterator** : Reçoit le tableau `data` issu de la réponse JSON de JSearch et sépare chaque offre d'emploi en un bundle individuel pour le traitement séquentiel.
4. **Tools / Set Multiple Variables** (ou Data Parser/Formatter) : Formate la date de publication au format JJ/MM/AAAA (ex: `formatDate(date; DD/MM/YYYY)`), et applique `ifempty()` pour les champs potentiellement nuls.
5. **Google Sheets - Add a Row** : Insère chaque offre nettoyée dans le tableau de suivi.
6. **Discord - Create a Message** : Envoie une alerte mise en forme dans le salon Discord dédié (titre du poste, entreprise, lieu, contrat et lien cliquable).

## D. Critères de recherche choisis
* **Mots-clés (query)** : Développeur Web *(à adapter selon le profil)*
* **Pays / Ville (country, location)** : FR, Paris *(ou autre ville)*
* **Période (date_posted)** : week (pour récupérer uniquement les offres des 7 derniers jours)

## E. Estimation des crédits Make & requêtes JSearch par mois
* **Planification** : 1 exécution par semaine = ~4 à 5 exécutions par mois.
* **Consommation Make** : ~6 opérations par exécution (si 10 offres traitées par exécution = 1 + 1 + 10x2 = ~22 opérations/semaine) -> **~90 à 100 opérations Make/mois** (largement en dessous de la limite gratuite de 1000).
* **Requêtes JSearch** : 1 requête HTTP par exécution -> **4 à 5 requêtes JSearch/mois** (limite gratuite RapidAPI JSearch : 500 requêtes/mois).

## F. Difficultés rencontrées et solutions
* **Champs manquants (Entreprise / Ville)** : Remplacement des valeurs vides `null` par une valeur par défaut (« Non précisé ») à l'aide de la fonction `ifempty(champ; "Non précisé")`.
* **Formatage des dates ISO** : Conversion des timestamps/ISO 8601 en JJ/MM/AAAA via `formatDate()`.
* **Sécurité des clés** : Nettoyage systématique de la clé API et du Webhook Discord avant l'export du blueprint JSON.

---

## G. Préparation aux questions de l'évaluation

**Q1 : À quoi sert le header `x-api-key` et pourquoi doit-il rester secret ?**
> **Réponse :** Le header `x-api-key` transmet la clé d'authentification à l'API JSearch (via RapidAPI) pour identifier et autoriser la requête. Elle doit rester secrète car elle est rattachée à votre compte personnel : si quelqu'un la récupère, il peut consommer votre quota de requêtes ou engendrer des frais à votre place.

**Q2 : Que se passerait-il sans l'utilisation de `ifempty` ?**
> **Réponse :** Sans `ifempty(valeur; "Valeur par défaut")`, si une offre renvoie une valeur null ou indéfinie pour le nom de l'entreprise ou le lieu, la cellule correspondante dans Google Sheets resterait vide ou afficherait une erreur, ce qui rompt le critère de propreté des données (C7). `ifempty` garantit un affichage propre (ex: "Non spécifié").

**Q3 : Comment modifier le scénario pour chercher dans une autre ville ou filtrer un seul type de contrat ?**
> **Réponse :**
> * **Autre ville :** Modifier le paramètre `query` ou la localisation dans le module HTTP JSearch (ex: remplacer "Développeur Web Paris" par "Développeur Web Lyon").
> * **Un seul type de contrat :**
>   1. Ajouter le paramètre `employment_types` dans l'API JSearch si supporté (ex: `FULLTIME`).
>   2. Ou ajouter un Filtre Make entre l'Iterator et le module Google Sheets avec la condition : `job_employment_type` contient `FULLTIME` (ou CDI).