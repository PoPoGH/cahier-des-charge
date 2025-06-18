# Cahier des charges - Système de donjon instancié
## Serveur Minecraft SAO France

---

## 1. Contexte et objectifs

### 1.1 Contexte
Les donjons d'Aincrad sont des zones dangereuses remplies de monstres et de trésors. Ce système doit créer des instances privées pour les joueurs, permettant une expérience de dungeon crawling authentique avec progression et récompenses.

### 1.2 Objectifs
- Créer des donjons instanciés pour groupes de joueurs
- Implémenter un système de progression de difficulté
- Récompenser les joueurs avec du butin adapté à leur niveau
- Générer des donjons procéduraux ou prédéfinis

---

## 2. Spécifications fonctionnelles

### 2.1 Système d'instances

#### 2.1.1 Création d'instances
- **Accès contrôlé** : Portails d'entrée avec conditions
- **Instances privées** : Une copie par groupe/joueur
- **Durée limitée** : 2 heures maximum par session
- **Sauvegarde** : État persistant en cas de déconnexion

#### 2.1.2 Types de donjons
- **Donjons de niveau** : Adaptés aux étages d'Aincrad (1-100)
- **Donjons thématiques** : Biomes spéciaux (Ice, Fire, Shadow)
- **Raids** : Donjons pour 10-20 joueurs
- **Donjons solo** : Défis individuels

#### 2.1.3 Mécaniques de jeu
- **Objectifs variés** : Élimination, collecte, survie, puzzle
- **Boss uniques** : Mécaniques spéciales par étage
- **Traps et pièges** : Mécanismes interactifs
- **Checkpoints** : Points de sauvegarde dans les longs donjons

### 2.2 Système de difficulté

#### 2.2.1 Scaling dynamique
- **Niveau du groupe** : Adaptation selon le niveau moyen
- **Nombre de joueurs** : HP et dégâts des mobs ajustés
- **Équipement** : Prise en compte de la puissance d'équipe
- **Difficulté sélectionnable** : Normal, Difficile, Cauchemar

#### 2.2.2 Mécaniques progressives
- **Paliers de difficulté** : Augmentation tous les 10 étages
- **Nouveaux mobs** : Introduction progressive d'ennemis
- **Mécaniques complexes** : Puzzles et défis évolutifs
- **Résistances** : Immunités et faiblesses variables

---

## 3. Spécifications techniques

### 3.1 Architecture des instances

#### 3.1.1 Gestion des mondes
```java
public class DungeonInstance {
    private String instanceId;
    private World instanceWorld;
    private Set<UUID> participants;
    private DungeonTemplate template;
    private DifficultyLevel difficulty;
    private long startTime;
    private InstanceState state;
}
```

#### 3.1.2 Templates de donjons
```yaml
# Exemple de template
dungeon_template:
  id: "aincrad_floor_1"
  name: "Plaines de départ"
  level_range: [1, 10]
  max_players: 6
  duration_minutes: 60
  schematic_file: "floor1_dungeon.schem"
  spawn_points:
    - x: 0, y: 64, z: 0
  boss_room:
    location: x: 100, y: 64, z: 100
    boss_type: "ILLFANG_THE_KOBOLD_LORD"
  loot_tables:
    - "aincrad_floor1_common"
    - "aincrad_floor1_rare"
```

### 3.2 Base de données

#### 3.2.1 Table Instances
```sql
CREATE TABLE donjon_instances (
    id VARCHAR(36) PRIMARY KEY,
    template_id VARCHAR(50) NOT NULL,
    leader_uuid VARCHAR(36) NOT NULL,
    participants JSON NOT NULL,
    difficulty ENUM('normal', 'difficile', 'cauchemar') DEFAULT 'normal',
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    end_time TIMESTAMP NULL,
    status ENUM('active', 'completed', 'failed', 'expired') DEFAULT 'active',
    current_checkpoint INT DEFAULT 0,
    boss_defeated BOOLEAN DEFAULT FALSE,
    monde_instance VARCHAR(100) NOT NULL
);
```

#### 3.2.2 Table Progressions
```sql
CREATE TABLE donjon_progression (
    id INT PRIMARY KEY AUTO_INCREMENT,
    joueur_uuid VARCHAR(36) NOT NULL,
    template_id VARCHAR(50) NOT NULL,
    best_time INT DEFAULT NULL,
    completions INT DEFAULT 0,
    best_difficulty ENUM('normal', 'difficile', 'cauchemar') DEFAULT 'normal',
    first_completion TIMESTAMP NULL,
    last_completion TIMESTAMP NULL
);
```

---

## 4. Système de récompenses

### 4.1 Loot dynamique

#### 4.1.1 Tables de butin
- **Qualité par difficulté** : Meilleur loot en mode difficile
- **Butin garanti** : Récompenses minimales assurées
- **Bonus de groupe** : Multiplicateurs selon la taille d'équipe
- **Butin de boss** : Objets uniques et rares

#### 4.1.2 Types de récompenses
- **Équipements** : Armes et armures d'Aincrad
- **Matériaux rares** : Composants de craft exclusifs
- **Cols** : Monnaie proportionnelle à la difficulté
- **Cristaux** : Objets consommables SAO
- **Titres** : Récompenses cosmétiques

### 4.2 Système d'expérience
- **XP de donjon** : Bonus d'expérience conséquent
- **XP de boss** : Récompense majeure pour les boss
- **Bonus de performance** : XP supplémentaire selon le temps
- **Malus de mort** : Pénalité en cas d'échec

---

## 5. Interface utilisateur

### 5.1 Commandes principales
- `/donjon list` - Liste des donjons disponibles
- `/donjon entrer <nom> [difficulté]` - Rejoindre un donjon
- `/donjon creer <template> <difficulté>` - Créer une instance
- `/donjon quitter` - Abandonner le donjon actuel
- `/donjon progression` - Voir ses statistiques
- `/donjon classement <template>` - Leaderboard

### 5.2 Interfaces graphiques
- **Sélecteur de donjon** : GUI avec preview et infos
- **Panneau de progression** : Carte du donjon avec objectifs
- **Inventaire de groupe** : Partage du butin trouvé
- **Statistiques live** : DPS, soins, dégâts subis

### 5.3 Système de notifications
- **Objectifs** : Progression des quêtes du donjon
- **Dangers** : Alertes de pièges ou boss
- **Récompenses** : Annonce du butin obtenu
- **Temps** : Countdown avant expiration

---

## 6. Mécaniques avancées

### 6.1 Boss et mécaniques spéciales

#### 6.1.1 IA des boss
- **Phases de combat** : Changements de comportement selon les HP
- **Mécaniques uniques** : Sorts et capacités spéciales
- **Patterns d'attaque** : Séquences prévisibles et aléatoires
- **Conditions de victoire** : Objectifs variés selon le boss

#### 6.1.2 Événements scriptés
- **Cinématiques** : Séquences d'introduction de boss
- **Événements temporisés** : Défis avec limite de temps
- **Interactions** : Leviers, portes, énigmes
- **Variations aléatoires** : Chemins et événements alternatifs

### 6.2 Donjons procéduraux
- **Génération de layouts** : Salles et couloirs aléatoires
- **Placement intelligent** : Mobs et trésors équilibrés
- **Thèmes cohérents** : Biomes et ambiances respectés
- **Rejouabilité** : Nouvelles expériences à chaque run

---

## 7. Optimisations et performance

### 7.1 Gestion mémoire
- **Limite d'instances** : Maximum 50 instances simultanées
- **Garbage collection** : Nettoyage automatique des instances
- **Préchargement** : Templates en cache pour accès rapide
- **Compression** : Sauvegardes optimisées

### 7.2 Limitation des ressources
- **Timeout automatique** : Fermeture des instances inactives
- **Limite par joueur** : Une instance active maximum
- **Cooldown de création** : 5 minutes entre deux instances
- **Monitoring** : Surveillance des performances serveur

---

## 8. Intégrations système

### 8.1 Plugins compatibles
- **MCCustomMobs** : Mobs SAO personnalisés
- **MythicItems** : Équipements et objets uniques
- **WorldEdit** : Chargement des templates de donjons
- **PlaceholderAPI** : Affichage de statistiques

### 8.2 API pour développeurs
```java
public interface DungeonAPI {
    void createInstance(Player leader, String templateId, DifficultyLevel difficulty);
    void addParticipant(String instanceId, Player player);
    void completeObjective(String instanceId, String objectiveId);
    void awardLoot(String instanceId, LootTable table);
}
```

---

## 9. Critères d'évaluation

### 9.1 Fonctionnalités obligatoires
- [ ] Création d'instances fonctionnelle
- [ ] Système de difficulté adaptatif
- [ ] Mécaniques de boss basiques
- [ ] Système de récompenses
- [ ] Interface utilisateur complète

### 9.2 Fonctionnalités bonus
- [ ] Génération procédurale
- [ ] Mécaniques de boss avancées
- [ ] Système de classements
- [ ] Donjons multi-étages
- [ ] API externe

### 9.3 Critères techniques
- [ ] Gestion optimisée des instances
- [ ] Performances stables avec 20+ instances
- [ ] Système de sauvegarde robuste
- [ ] Interface intuitive et responsive
- [ ] Code modulaire et extensible

---

## 10. Planning de développement

| Phase | Durée | Description |
|-------|-------|-------------|
| Conception technique | 2 jours | Architecture et templates |
| Système d'instances | 4 jours | Création et gestion des mondes |
| Mécaniques de jeu | 4 jours | Objectifs, boss, progression |
| Système de loot | 2 jours | Récompenses et butin |
| Interface utilisateur | 3 jours | GUI et commandes |
| Optimisations | 2 jours | Performance et stabilité |
| Tests et polish | 2 jours | Tests de charge et finitions |

**Total estimé : 19 jours**

---

*Cahier des charges créé pour le recrutement développeur SAO France - 18 juin 2025*
