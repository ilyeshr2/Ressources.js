# Application de Gestion des Ressources

Cette application Vue.js permet aux utilisateurs de gérer une liste de ressources (livres, articles, vidéos, etc.). Elle offre des fonctionnalités de filtrage, de recherche, de tri, de pagination et de thèmes clairs et sombres.

## Fonctionnalités Principales

### 1. Thème Sombre / Clair
- **Commutateur de thème** : Permet à l'utilisateur de basculer entre un thème clair et un thème sombre, le choix est sauvegardé dans `localStorage`.

### 2. Section Statistiques
- **Total des ressources** : Affiche le nombre total de ressources enregistrées.
- **Ressources par type** : Affiche un décompte des ressources regroupées par type (Livre, Article, Vidéo, etc.).
- **Dernière modification** : Affiche la date et l'heure de la dernière modification de la liste de ressources.
- **Graphique des ressources** : Un graphique en barres (généré avec Chart.js) montre la distribution des ressources par type.

### 3. Formulaire d'Ajout / Modification de Ressource
- Formulaire complet pour ajouter une nouvelle ressource ou modifier une ressource existante.
- Champs pour le nom, le type, la description, les tags et la priorité (Basse, Moyenne, Haute).
- Validation basique avec affichage d'erreurs pour les champs requis.

### 4. Recherche, Filtrage et Tri
- **Recherche** : Recherche textuelle basée sur le nom, le type, et la description.
- **Filtrage** : Filtre les ressources par type sélectionné.
- **Tri** : Trie les ressources par nom, type, date de création, ou priorité.

### 5. Liste de Ressources
- Affiche les ressources sous forme de liste, avec des actions telles que l'édition, la suppression, et l'ajout aux favoris.
- Les ressources sont paginées, avec une navigation "Précédent" et "Suivant".

### 6. Notifications
- Affiche des messages de notification (toasts) pour informer l'utilisateur des actions réussies (ajout, modification, suppression, etc.).

## Structure des Données

Chaque ressource est un objet contenant les propriétés suivantes :
- `id` : Identifiant unique généré lors de l'ajout.
- `nom` : Nom de la ressource.
- `type` : Type de ressource (Livre, Article, Vidéo, Podcast, ou Cours).
- `description` : Description de la ressource.
- `tags` : Mots-clés associés, séparés par des virgules.
- `priority` : Priorité (basse, moyenne, haute).
- `dateCreation` : Date de création de la ressource.
- `favorite` : Statut de favori.

## Installation

1. **Clonez ce dépôt** :
   ```bash
   git clone https://github.com/ilyeshr2/Ressources.js
   cd Ressources.js
   ```

2. **Installez les dépendances** :
   ```bash
   npm install
   ```

3. **Lancez le serveur de développement** :
   ```bash
   npm run serve
   ```

4. **Accédez à l'application** :
   Ouvrez `http://localhost:8080` dans votre navigateur.

## Utilisation

1. **Ajoutez des Ressources** :
   Remplissez le formulaire et cliquez sur le bouton "Ajouter Ressource". Les ressources sont sauvegardées dans le `localStorage`.

2. **Modifier une Ressource** :
   Cliquez sur le bouton "Éditer" de la ressource. Le formulaire sera prérempli avec les informations de la ressource sélectionnée.

3. **Supprimer une Ressource** :
   Cliquez sur le bouton "Supprimer" pour retirer une ressource de la liste. Une confirmation vous sera demandée.

4. **Pagination** :
   Naviguez entre les pages de ressources en utilisant les boutons "Précédent" et "Suivant".

5. **Filtrer et Trier** :
   Utilisez les champs de filtrage et de tri pour trouver et organiser les ressources.

6. **Changer de Thème** :
   Utilisez le bouton de thème pour basculer entre le mode clair et sombre.

## Technologies Utilisées

- **Vue.js** : Framework JavaScript pour construire l'interface utilisateur.
- **Chart.js** : Bibliothèque pour créer le graphique de statistiques.
- **LocalStorage** : Utilisé pour stocker les ressources et conserver le thème et la dernière modification entre les sessions.
- **CSS** : Pour styliser l'application, y compris des styles spécifiques pour le thème sombre et clair.

## Personnalisation

- **Nombre de ressources par page** : Modifiez la variable `parPage` pour ajuster le nombre de ressources affichées par page.
- **Types de ressources** : Les types disponibles dans le formulaire peuvent être personnalisés dans l'élément `<select>` de la propriété `type`.

## Dépendances

- **Vue.js** : ^2.6.12
- **Chart.js** : ^3.0.0

