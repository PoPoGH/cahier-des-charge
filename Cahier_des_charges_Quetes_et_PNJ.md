# Cahier des charges - Système de quêtes et PNJ SAO
## Serveur Minecraft SAO France

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans l'univers de SAO, les PNJ proposent des quêtes variées pour aider les joueurs à progresser et découvrir l'histoire d'Aincrad. Ce système doit offrir une expérience immersive avec des dialogues riches et des quêtes dynamiques.

### 1.2 Objectifs
- Créer un système de quêtes immersif et évolutif
- Implémenter des PNJ avec intelligence artificielle basique
- Proposer des récompenses attractives et équilibrées
- Générer du contenu rejouable et évolutif

---

## 2. Spécifications fonctionnelles

### 2.1 Système de PNJ

#### 2.1.1 Types de PNJ
- **Marchands** : Vente d'objets et équipements
- **Questgivers** : Donneurs de quêtes principales et secondaires
- **Gardes** : Sécurité des villes et informations
- **Artisans** : Services de craft et amélioration
- **PNJ informatifs** : Lore et histoire d'Aincrad

#### 2.1.2 Comportements des PNJ
- **Dialogues contextuels** : Réponses selon la progression du joueur
- **Horaires d'activité** : Déplacements et disponibilité variables
- **Relations dynamiques** : Réputation avec chaque PNJ
- **Réactions aux événements** : Comportement selon les actions du serveur

#### 2.1.3 Intelligence artificielle
- **Pathfinding** : Déplacement intelligent dans l'environnement
- **Détection de joueurs** : Reconnaissance et salutation
- **États émotionnels** : Humeur variable selon les interactions
- **Mémoire des interactions** : Historique des échanges

### 2.2 Système de quêtes

#### 2.2.1 Types de quêtes
- **Quêtes principales** : Histoire principale d'Aincrad
- **Quêtes secondaires** : Exploration et développement
- **Quêtes journalières** : Contenu répétable avec récompenses
- **Quêtes de guilde** : Missions coopératives
- **Quêtes événementielles** : Contenu temporaire spécial

#### 2.2.2 Mécaniques de quêtes
- **Objectifs variés** : Collecte, élimination, exploration, livraison
- **Conditions de déblocage** : Niveau, quêtes précédentes, réputation
- **Chaînes de quêtes** : Séries de missions liées
- **Embranchements** : Choix modifiant le déroulement

---

## 3. Spécifications techniques

### 3.1 Architecture des données

#### 3.1.1 Base de données PNJ
```sql
CREATE TABLE npcs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    uuid VARCHAR(36) UNIQUE NOT NULL,
    nom VARCHAR(100) NOT NULL,
    type ENUM('marchand', 'questgiver', 'garde', 'artisan', 'informatif') NOT NULL,
    monde VARCHAR(50) NOT NULL,
    x DOUBLE NOT NULL,
    y DOUBLE NOT NULL,
    z DOUBLE NOT NULL,
    skin_texture TEXT,
    dialogues JSON,
    horaires JSON,
    actif BOOLEAN DEFAULT TRUE
);
```

#### 3.1.2 Base de données quêtes
```sql
CREATE TABLE quetes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(200) NOT NULL,
    description TEXT NOT NULL,
    type ENUM('principale', 'secondaire', 'journaliere', 'guilde', 'evenementielle') NOT NULL,
    pnj_donneur INT NOT NULL,
    niveau_requis INT DEFAULT 1,
    quetes_prerequis JSON,
    objectifs JSON NOT NULL,
    recompenses JSON NOT NULL,
    script_completion TEXT,
    actif BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (pnj_donneur) REFERENCES npcs(id)
);
```

#### 3.1.3 Progression des joueurs
```sql
CREATE TABLE joueur_quetes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    joueur_uuid VARCHAR(36) NOT NULL,
    quete_id INT NOT NULL,
    statut ENUM('disponible', 'en_cours', 'terminee', 'echouee') DEFAULT 'disponible',
    progression JSON,
    date_debut TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    date_fin TIMESTAMP NULL,
    FOREIGN KEY (quete_id) REFERENCES quetes(id)
);

CREATE TABLE joueur_reputation (
    joueur_uuid VARCHAR(36) NOT NULL,
    pnj_id INT NOT NULL,
    reputation INT DEFAULT 0,
    derniere_interaction TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (joueur_uuid, pnj_id),
    FOREIGN KEY (pnj_id) REFERENCES npcs(id)
);
```

### 3.2 Classes Java principales

#### 3.2.1 Système de PNJ
```java
public class CustomNPC {
    private String uuid;
    private String nom;
    private NPCType type;
    private Location position;
    private Map<String, Object> dialogues;
    private Schedule horaires;
    private Map<UUID, Integer> reputation;
    
    public void interact(Player player) { /* ... */ }
    public void updateBehavior() { /* ... */ }
}

public class NPCManager {
    private Map<String, CustomNPC> npcs;
    
    public void spawnNPC(CustomNPC npc) { /* ... */ }
    public void updateAllNPCs() { /* ... */ }
    public CustomNPC getNearestNPC(Location loc, double radius) { /* ... */ }
}
```

#### 3.2.2 Système de quêtes
```java
public class Quest {
    private int id;
    private String nom;
    private String description;
    private QuestType type;
    private List<QuestObjective> objectifs;
    private Map<String, Object> recompenses;
    
    public boolean canStart(Player player) { /* ... */ }
    public void complete(Player player) { /* ... */ }
}

public class QuestManager {
    private Map<UUID, List<Quest>> activeQuests;
    
    public void startQuest(Player player, Quest quest) { /* ... */ }
    public void updateProgress(Player player, String objectiveKey, Object value) { /* ... */ }
}
```

---

## 4. Interface utilisateur

### 4.1 Système de dialogues

#### 4.1.1 Interface de conversation
- **Arbre de dialogue** : Choix multiples et embranchements
- **Texte formaté** : Couleurs et styles selon le contexte
- **Portraits de PNJ** : Têtes et expressions émotionnelles
- **Options contextuelles** : Choix selon la progression et réputation

#### 4.1.2 Exemple de dialogue
```yaml
dialogue_marchand_armes:
  entree: "Bienvenue dans ma forge, aventurier !"
  options:
    - text: "Montrez-moi vos armes"
      action: "open_shop"
      condition: "reputation >= 0"
    - text: "Avez-vous des quêtes pour moi ?"
      action: "show_quests"
      condition: "level >= 10"
    - text: "Au revoir"
      action: "close_dialogue"
```

### 4.2 Interface de quêtes

#### 4.2.1 Journal de quêtes
- **Liste organisée** : Tri par type et statut
- **Suivi de progression** : Barres de progression et objectifs
- **Descriptions détaillées** : Lore et instructions complètes
- **Système de favoris** : Épinglage de quêtes importantes

#### 4.2.2 Commandes principales
- `/quete list` - Afficher les quêtes actives
- `/quete info <id>` - Détails d'une quête
- `/quete abandonner <id>` - Abandonner une quête
- `/quete journal` - Ouvrir l'interface graphique
- `/reputation <pnj>` - Voir sa réputation avec un PNJ

### 4.3 Indicateurs visuels
- **Marqueurs au-dessus des PNJ** : Icônes de quêtes disponibles
- **Waypoints** : Points de repère pour les objectifs
- **Notifications** : Alertes de progression et completion
- **Effets de particules** : Feedback visuel des interactions

---

## 5. Contenu des quêtes

### 5.1 Exemples de quêtes principales

#### 5.1.1 "Premier pas dans Aincrad" (Niveau 1-5)
```yaml
quete_tutorial:
  nom: "Premier pas dans Aincrad"
  description: "Apprenez les bases de la survie dans ce monde virtuel."
  objectifs:
    - type: "parler_pnj"
      target: "guide_klein"
      description: "Parler au guide Klein"
    - type: "tuer_mobs"
      target: "wild_boar"
      quantite: 5
      description: "Éliminer 5 sangliers sauvages"
    - type: "collecter_items"
      items: ["col:100"]
      description: "Collecter 100 cols"
  recompenses:
    experience: 500
    cols: 250
    items: ["starter_sword", "leather_armor"]
```

#### 5.1.2 "Le mystère du premier étage" (Niveau 8-12)
```yaml
quete_mystere_etage1:
  nom: "Le mystère du premier étage"
  description: "Des disparitions étranges ont lieu près de la forêt de l'ouest."
  objectifs:
    - type: "explorer_zone"
      zone: "foret_ouest"
      description: "Explorer la forêt de l'ouest"
    - type: "trouver_indices"
      indices: ["empreinte_mysterieuse", "tissu_dechire", "message_crypte"]
      description: "Trouver des indices sur les disparitions"
    - type: "vaincre_boss"
      boss: "alpha_wolf"
      description: "Vaincre le loup alpha responsable"
  recompenses:
    experience: 1500
    cols: 500
    titre: "Enquêteur d'Aincrad"
```

### 5.2 Quêtes journalières et événementielles

#### 5.2.1 Rotation quotidienne
- **Chasse aux monstres** : Élimination de créatures spécifiques
- **Collecte de ressources** : Rassemblement de matériaux rares
- **Livraisons** : Transport d'objets entre PNJ
- **Exploration** : Découverte de zones cachées

#### 5.2.2 Événements spéciaux
- **Festival de la moisson** : Quêtes agricoles avec récompenses saisonnières
- **Tournoi des étages** : Compétitions entre joueurs
- **Invasion de boss** : Événements de serveur avec boss géants
- **Mystères d'Aincrad** : Enquêtes temporaires avec énigmes

---

## 6. Système de récompenses

### 6.1 Types de récompenses

#### 6.1.1 Récompenses classiques
- **Expérience** : Progression de niveau du joueur
- **Cols** : Monnaie du serveur pour achats
- **Objets** : Équipements, consommables, matériaux
- **Compétences** : Points de compétence ou déblocages

#### 6.1.2 Récompenses spéciales
- **Titres** : Distinctions cosmétiques uniques
- **Montures** : Moyens de transport exclusifs
- **Familiers** : Compagnons de combat
- **Accès à zones** : Déblocage de nouvelles régions

### 6.2 Système de réputation
- **Niveaux de réputation** : Hostile (-1000) à Héroïque (+1000)
- **Bonus par niveau** : Réductions prix, quêtes exclusives
- **Mécaniques d'influence** : Actions affectant la réputation globale
- **Factions** : Groupes de PNJ avec réputations liées

---

## 7. Mécaniques avancées

### 7.1 Génération procédurale

#### 7.1.1 Quêtes dynamiques
- **Objectifs variables** : Adaptation selon le niveau du joueur
- **Lieux aléatoires** : Génération de points d'intérêt temporaires
- **Récompenses fluctuantes** : Adaptation selon l'économie serveur
- **Conditions météo** : Quêtes influencées par l'environnement

#### 7.1.2 Événements émergents
- **Attaques de monstres** : Menaces temporaires sur les villes
- **Caravanes** : Escortes commerciales avec PNJ voyageurs
- **Découvertes** : Apparition de donjons ou trésors cachés
- **Rencontres aléatoires** : PNJ temporaires avec missions courtes

### 7.2 Intelligence sociale des PNJ
- **Relations entre PNJ** : Réseaux sociaux influençant les quêtes
- **Rumeurs et informations** : Propagation de nouvelles entre PNJ
- **Économie dynamique** : Prix fluctuants selon l'offre/demande
- **Personnalités uniques** : Traits de caractère affectant les interactions

---

## 8. Intégrations système

### 8.1 Compatibilité plugins
- **Citizens2** : Gestion avancée des PNJ
- **MythicMobs** : Monstres personnalisés pour les quêtes
- **Vault** : Économie et permissions
- **WorldGuard** : Zones et protection des PNJ
- **PlaceholderAPI** : Affichage d'informations dynamiques

### 8.2 API pour développeurs
```java
public interface QuestAPI {
    void registerQuest(Quest quest);
    void createNPC(NPCData data);
    void updateQuestProgress(Player player, String questId, String objective, Object value);
    int getPlayerReputation(Player player, String npcId);
}
```

---

## 9. Critères d'évaluation

### 9.1 Fonctionnalités obligatoires
- [ ] Système de PNJ fonctionnel avec dialogues
- [ ] Création et gestion de quêtes
- [ ] Interface de journal de quêtes
- [ ] Système de récompenses opérationnel
- [ ] Sauvegarde de progression

### 9.2 Fonctionnalités bonus
- [ ] Intelligence artificielle des PNJ
- [ ] Génération de quêtes dynamiques
- [ ] Système de réputation avancé
- [ ] Événements émergents
- [ ] Intégration avec autres plugins

### 9.3 Critères techniques
- [ ] Performance avec 100+ PNJ actifs
- [ ] Interface utilisateur intuitive
- [ ] Système de scripts pour quêtes
- [ ] Équilibrage des récompenses
- [ ] Code modulaire et extensible

---

## 10. Planning de développement

| Phase | Durée | Description |
|-------|-------|-------------|
| Architecture système | 3 jours | BDD, classes principales, API |
| Système de PNJ | 5 jours | Spawn, comportement, dialogues |
| Système de quêtes | 4 jours | Création, progression, objectifs |
| Interface utilisateur | 4 jours | GUI journal, dialogues, notifications |
| Système de récompenses | 2 jours | Distribution, équilibrage |
| Contenu initial | 3 jours | PNJ et quêtes de base |
| Tests et optimisation | 2 jours | Performance, bugs, polish |

**Total estimé : 23 jours**

---

*Cahier des charges créé pour le recrutement développeur SAO France - 18 juin 2025*
