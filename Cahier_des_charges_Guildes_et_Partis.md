# Cahier des charges - Système de guildes et partis
## Serveur Minecraft SAO France

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans Sword Art Online, les joueurs se regroupent en guildes et forment des partis temporaires pour affronter les dangers d'Aincrad. Ce système doit reproduire fidèlement l'expérience de coopération et d'organisation des joueurs.

### 1.2 Objectifs
- Créer un système de guildes persistantes avec hiérarchie
- Permettre la formation de partis temporaires pour les donjons
- Implémenter un système de partage d'expérience et de butin
- Favoriser l'entraide et la stratégie entre joueurs

---

## 2. Spécifications fonctionnelles

### 2.1 Système de guildes

#### 2.1.1 Création et gestion
- **Création** : Coût de création (10 000 cols)
- **Nom unique** : Vérification de l'unicité des noms de guilde
- **Emblème** : Système de bannières personnalisables
- **Niveau de guilde** : Progression basée sur les activités des membres

#### 2.1.2 Hiérarchie
- **Maître de Guilde** : Contrôle total, transfert possible
- **Officiers** : Invitation/exclusion, gestion des permissions
- **Membres vétérans** : Privilèges étendus après ancienneté
- **Membres** : Droits de base
- **Recrues** : Période d'essai de 7 jours

#### 2.1.3 Fonctionnalités avancées
- **Base de guilde** : Zone protégée avec téléportation
- **Coffre-fort** : Stockage partagé avec logs
- **Panneau d'affichage** : Messages et annonces internes
- **Système d'alliances** : Pactes entre guildes

### 2.2 Système de partis

#### 2.2.1 Formation
- **Création rapide** : Invitation par proximité ou commande
- **Limite** : Maximum 6 joueurs par parti (comme dans SAO)
- **Rôles** : Leader, membres, système de vote
- **Durée** : Dissolution automatique après déconnexion du leader

#### 2.2.2 Avantages
- **Partage XP** : Distribution équitable dans un rayon de 50 blocs
- **Partage butin** : Options de répartition automatique/manuelle
- **Communication** : Chat privé du parti
- **Téléportation** : TP vers les membres (avec limitations)

---

## 3. Spécifications techniques

### 3.1 Architecture de données

#### 3.1.1 Table Guildes
```sql
CREATE TABLE guildes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(50) UNIQUE NOT NULL,
    maitre_uuid VARCHAR(36) NOT NULL,
    creation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    niveau INT DEFAULT 1,
    experience BIGINT DEFAULT 0,
    cols DECIMAL(15,2) DEFAULT 0,
    description TEXT,
    embleme TEXT,
    base_monde VARCHAR(50),
    base_x INT,
    base_y INT,
    base_z INT
);
```

#### 3.1.2 Table Membres
```sql
CREATE TABLE guilde_membres (
    id INT PRIMARY KEY AUTO_INCREMENT,
    guilde_id INT NOT NULL,
    joueur_uuid VARCHAR(36) NOT NULL,
    rang ENUM('maitre', 'officier', 'veteran', 'membre', 'recrue') DEFAULT 'recrue',
    date_adhesion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    contribution_xp BIGINT DEFAULT 0,
    contribution_cols DECIMAL(15,2) DEFAULT 0,
    statut ENUM('actif', 'inactif', 'banni') DEFAULT 'actif',
    FOREIGN KEY (guilde_id) REFERENCES guildes(id)
);
```

### 3.2 Système de partis en mémoire
```java
public class Parti {
    private UUID leaderId;
    private Set<UUID> membres;
    private PartiConfig config;
    private long creationTime;
    private Location rallyPoint;
}
```

---

## 4. Interface utilisateur

### 4.1 Commandes guildes
- `/guilde creer <nom>` - Créer une nouvelle guilde
- `/guilde info [guilde]` - Informations sur la guilde
- `/guilde inviter <joueur>` - Inviter un joueur
- `/guilde exclure <joueur>` - Exclure un membre
- `/guilde promouvoir <joueur>` - Changer le rang
- `/guilde base` - Téléportation à la base
- `/guilde chat <message>` - Chat de guilde

### 4.2 Commandes partis
- `/parti creer` - Créer un nouveau parti
- `/parti inviter <joueur>` - Inviter dans le parti
- `/parti quitter` - Quitter le parti actuel
- `/parti leader <joueur>` - Transférer le leadership
- `/parti tp <joueur>` - Se téléporter vers un membre
- `/parti config` - Configuration du partage

### 4.3 Menus GUI
- **Menu guilde** : Gestion complète via interface graphique
- **Liste des membres** : Statuts, rangs, contributions
- **Coffre-fort** : Interface sécurisée avec logs
- **Configuration parti** : Options de partage XP/butin

---

## 5. Fonctionnalités avancées

### 5.1 Système de guerres de guildes
- **Déclaration** : Procédure formelle avec conditions
- **Zones de conflit** : Territoires disputés
- **Objectifs** : Capture de points stratégiques
- **Récompenses** : Cols, territoires, prestige

### 5.2 Événements et raids
- **Boss de guilde** : Défis exclusifs aux guildes
- **Donjons de parti** : Instances pour groupes organisés
- **Tournois** : Compétitions inter-guildes
- **Quêtes collectives** : Missions nécessitant coopération

### 5.3 Économie intégrée
- **Taxes de guilde** : Contribution automatique optionnelle
- **Marché interne** : Échanges entre membres
- **Investissements** : Amélioration de la base commune
- **Contrats** : Missions payées par la guilde

---

## 6. Critères d'évaluation

### 6.1 Fonctionnalités obligatoires
- [ ] Système de guildes complet avec hiérarchie
- [ ] Formation et gestion de partis
- [ ] Partage d'expérience fonctionnel
- [ ] Interface utilisateur intuitive
- [ ] Persistance des données

### 6.2 Fonctionnalités bonus
- [ ] Système de guerres de guildes
- [ ] Événements spéciaux
- [ ] API pour autres plugins
- [ ] Statistiques avancées
- [ ] Integration Discord

### 6.3 Critères techniques
- [ ] Performance avec 100+ guildes actives
- [ ] Gestion des permissions avancée
- [ ] Système de backup automatique
- [ ] Logs détaillés des actions
- [ ] Configuration flexible

---

## 7. Règles de gestion

### 7.1 Limitations
- Maximum 50 membres par guilde
- Une seule guilde par joueur
- Un seul parti actif par joueur
- Cooldown de 24h pour changer de guilde

### 7.2 Progression
- **Niveau guilde** : Basé sur XP collective des membres
- **Avantages par niveau** : Bonus XP, capacité membre, etc.
- **Prestige** : Classement serveur des guildes

### 7.3 Sanctions
- **Exclusion** : Cooldown avant nouvelle adhésion
- **Dissolution** : Procédure avec vote des officiers
- **Sanctions automatiques** : Inactivité prolongée

---

## 8. Intégrations

### 8.1 Autres plugins
- **Économie** : Vault API pour la gestion des cols
- **Permissions** : LuckPerms pour les droits étendus
- **Protection** : WorldGuard pour les bases de guildes
- **Chat** : EssentialsX pour les canaux privés

### 8.2 Base de données
- **Performance** : Index optimisés pour les requêtes fréquentes
- **Sauvegarde** : Backup automatique toutes les heures
- **Migration** : Scripts de mise à jour de schéma

---

## 9. Planning de développement

| Phase | Durée | Description |
|-------|-------|-------------|
| Architecture | 2 jours | Conception technique et BDD |
| Core Guildes | 4 jours | Système de base des guildes |
| Système Partis | 2 jours | Gestion des partis temporaires |
| Interface GUI | 3 jours | Menus et interface utilisateur |
| Fonctionnalités avancées | 3 jours | Guerres, événements, économie |
| Tests et optimisation | 2 jours | Tests de charge et corrections |

**Total estimé : 16 jours**

---

*Cahier des charges créé pour le recrutement développeur SAO France - 18 juin 2025*
