# Cahier des charges - Système de location de chambres de taverne
## Serveur Minecraft SAO France

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans l'univers de Sword Art Online, les tavernes constituent des lieux de repos essentiels pour les joueurs. Ce système permettra aux joueurs de louer des chambres dans les tavernes pour y stocker leurs affaires et se reposer en sécurité.

### 1.2 Objectifs
- Créer un système immersif de location de chambres
- Permettre aux joueurs de disposer d'un espace privé temporaire
- Générer une économie autour de l'hébergement
- Offrir une alternative aux homes permanents

---

## 2. Spécifications fonctionnelles

### 2.1 Fonctionnalités principales

#### 2.1.1 Location de chambre
- **Durée** : Location par périodes (1 jour, 3 jours, 7 jours)
- **Paiement** : Système de paiement en monnaie du serveur
- **Accès exclusif** : Seul le locataire peut accéder à sa chambre
- **Stockage** : Coffres sécurisés dans la chambre

#### 2.1.2 Gestion des tavernes
- **Types de chambres** : Standard, Premium, Suite
- **Tarification** : Prix différents selon le type de chambre et la durée
- **Disponibilité** : Système de réservation temps réel

#### 2.1.3 Interface utilisateur
- **Menu interactif** : Interface graphique pour la location
- **Panneaux d'information** : Affichage des prix et disponibilités
- **Notifications** : Alertes avant expiration de la location

### 2.2 Fonctionnalités secondaires
- **Système de prolongation** : Possibilité d'étendre la location
- **Historique** : Suivi des locations précédentes
- **Système de parrainage** : Réductions pour les habitués

---

## 3. Spécifications techniques

### 3.1 Technologies requises
- **Plugin Minecraft** : Paper 1.20.3
- **Base de données** : MySQL pour la persistance
- **Économie** : Integration avec Vault API
- **Permissions** : Système de permissions compatible

### 3.2 Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Interface     │    │   Logique       │    │   Stockage      │
│   Utilisateur   │◄──►│   Métier        │◄──►│   Données       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
   - Menus GUI            - Gestion locations    - Base données
   - Commandes            - Calcul tarifs        - Fichiers config
   - Panneaux             - Contrôle accès       - Logs
```

### 3.3 Structures de données

#### 3.3.1 Table Tavernes
```sql
CREATE TABLE tavernes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100) NOT NULL,
    monde VARCHAR(50) NOT NULL,
    x INT NOT NULL,
    y INT NOT NULL,
    z INT NOT NULL,
    proprietaire VARCHAR(36),
    actif BOOLEAN DEFAULT TRUE
);
```

#### 3.3.2 Table Chambres
```sql
CREATE TABLE chambres (
    id INT PRIMARY KEY AUTO_INCREMENT,
    taverne_id INT NOT NULL,
    numero INT NOT NULL,
    type ENUM('standard', 'premium', 'suite') NOT NULL,
    prix_jour DECIMAL(10,2) NOT NULL,
    region_name VARCHAR(100) NOT NULL,
    FOREIGN KEY (taverne_id) REFERENCES tavernes(id)
);
```

#### 3.3.3 Table Locations
```sql
CREATE TABLE locations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    chambre_id INT NOT NULL,
    joueur_uuid VARCHAR(36) NOT NULL,
    date_debut TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    date_fin TIMESTAMP NOT NULL,
    prix_total DECIMAL(10,2) NOT NULL,
    statut ENUM('active', 'expiree', 'annulee') DEFAULT 'active',
    FOREIGN KEY (chambre_id) REFERENCES chambres(id)
);
```

---

## 4. Interface utilisateur

### 4.1 Commandes principales
- `/taverne list` - Afficher les tavernes disponibles
- `/taverne info <nom>` - Informations sur une taverne
- `/taverne louer <taverne> <durée>` - Louer une chambre
- `/taverne prolonger <durée>` - Prolonger la location actuelle
- `/taverne quitter` - Libérer la chambre actuelle

### 4.2 Menus GUI
- **Menu principal** : Liste des tavernes avec disponibilités
- **Menu taverne** : Chambres disponibles avec prix
- **Menu location** : Gestion de la location actuelle

### 4.3 Panneaux informatifs
- Panneau à l'entrée de chaque taverne
- Affichage des chambres libres/occupées
- Prix et promotions du moment

---

## 5. Règles de gestion

### 5.1 Tarification
- **Chambre standard** : 100 coins/jour
- **Chambre premium** : 200 coins/jour  
- **Suite** : 500 coins/jour
- **Réductions** : -10% pour 3 jours, -20% pour 7 jours

### 5.2 Restrictions
- Une seule location active par joueur
- Paiement obligatoire à l'avance
- Expulsion automatique en fin de location
- Sauvegarde des items dans l'inventaire du joueur

### 5.3 Permissions
- `taverne.user` - Utilisation basique
- `taverne.admin` - Administration des tavernes
- `taverne.premium` - Accès aux chambres premium

---

## 6. Critères d'évaluation

### 6.1 Fonctionnalités obligatoires
- [ ] Système de location fonctionnel
- [ ] Interface utilisateur intuitive
- [ ] Gestion des permissions et accès
- [ ] Persistance des données
- [ ] Integration économique

### 6.2 Fonctionnalités bonus
- [ ] Interface graphique avancée
- [ ] Système de notifications
- [ ] Statistiques et rapports
- [ ] API pour autres plugins
- [ ] Tests unitaires

### 6.3 Critères techniques
- [ ] Code propre et documenté
- [ ] Gestion des erreurs
- [ ] Performance optimisée
- [ ] Compatibilité serveur
- [ ] Configuration flexible

---

## 7. Livrables attendus

### 7.1 Code source
- Plugin complet avec toutes les fonctionnalités
- Fichiers de configuration
- Scripts de base de données
- Documentation technique

### 7.2 Documentation
- Guide d'installation
- Manuel utilisateur
- Documentation API
- Exemples de configuration

### 7.3 Tests
- Démonstration fonctionnelle
- Tests de charge basiques
- Scénarios d'utilisation

---

## 8. Planning suggéré

| Phase | Durée | Description |
|-------|-------|-------------|
| Analyse | 1 jour | Étude approfondie du cahier des charges |
| Architecture | 1 jour | Conception technique et base de données |
| Développement Core | 3 jours | Fonctionnalités principales |
| Interface | 2 jours | Menus et commandes |
| Tests | 1 jour | Tests et corrections |
| Documentation | 1 jour | Rédaction de la documentation |

**Total estimé : 9 jours**

---

## 9. Contact et questions

Pour toute question sur ce cahier des charges, veuillez contacter l'équipe de développement SAO France.

**Bonne chance pour votre candidature !**

---

*Document créé le 18 juin 2025 pour le recrutement développeur SAO France*
