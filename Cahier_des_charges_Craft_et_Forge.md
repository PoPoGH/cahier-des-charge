# Cahier des charges - Système de craft et forge SAO
## Serveur Minecraft SAO France

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans Sword Art Online, l'artisanat est une compétence essentielle permettant aux joueurs de créer leurs propres équipements. Le système doit reproduire fidèlement l'expérience de forge avec des mini-jeux, des recettes complexes et un système de qualité.

### 1.2 Objectifs
- Remplacer le craft vanilla par un système SAO immersif
- Implémenter des mini-jeux de forge interactifs
- Créer un système de qualité et de réussite/échec
- Proposer des recettes évolutives par niveau de compétence

---

## 2. Spécifications fonctionnelles

### 2.1 Système de compétences d'artisanat

#### 2.1.1 Compétences principales
- **Forge d'armes** : Épées, dagues, lances, masses
- **Forge d'armures** : Plastrons, casques, jambières, bottes
- **Alchimie** : Potions, cristaux, objets consommables
- **Cuisine** : Plats donnant des buffs temporaires
- **Joaillerie** : Accessoires et enchantements

#### 2.1.2 Progression des compétences
- **Niveaux** : 1-1000 comme dans SAO
- **Expérience** : Gain par craft réussi et qualité
- **Déblocage de recettes** : Nouvelles recettes selon le niveau
- **Spécialisations** : Branches dédiées à certains types d'objets

### 2.2 Mini-jeux de craft

#### 2.2.1 Mini-jeu de forge
- **Timing** : Séquences de clics au bon moment
- **Température** : Gestion de la chaleur du métal
- **Précision** : Zones de frappe précises
- **Endurance** : Gestion de l'énergie du joueur

#### 2.2.2 Mini-jeu d'alchimie
- **Dosage** : Quantités précises d'ingrédients
- **Ordre de mélange** : Séquence importante
- **Temps de cuisson** : Timing de chauffe/refroidissement
- **Stabilité** : Éviter la surchauffe/explosion

#### 2.2.3 Mini-jeu de cuisine
- **Découpe** : Préparation des ingrédients
- **Cuisson** : Contrôle de température et temps
- **Assaisonnement** : Équilibre des saveurs
- **Présentation** : Bonus esthétique

---

## 3. Système de qualité et matériaux

### 3.1 Qualité des objets craftés

#### 3.1.1 Niveaux de qualité
- **Défaillant** : -20% stats, couleur grise
- **Basique** : Stats normales, couleur blanche
- **Bien fait** : +10% stats, couleur verte
- **Excellent** : +25% stats, couleur bleue
- **Parfait** : +50% stats, couleur violette
- **Légendaire** : +100% stats, couleur orange

#### 3.1.2 Facteurs de qualité
- **Performance mini-jeu** : Score obtenu (0-100%)
- **Niveau de compétence** : Bonus selon la maîtrise
- **Qualité des matériaux** : Multiplicateur des composants
- **Équipement de forge** : Bonus d'outils avancés

### 3.2 Système de matériaux

#### 3.2.1 Types de matériaux
- **Métaux** : Fer, acier, mithril, adamantine
- **Bois** : Chêne, ébène, bois sacré, bois de dragon
- **Cuirs** : Cuir basique, cuir de monstre, cuir de boss
- **Gemmes** : Amélioration d'objets et enchantements
- **Composants rares** : Drops de boss et événements

#### 3.2.2 Propriétés des matériaux
```yaml
materiau_exemple:
  mithril:
    durabilitе: 2500
    bonus_attaque: 15
    bonus_defense: 10
    rarete: "rare"
    niveau_requis: 400
    couleur_item: "#87CEEB"
```

---

## 4. Spécifications techniques

### 4.1 Architecture du système

#### 4.1.1 Base de données
```sql
CREATE TABLE player_skills (
    joueur_uuid VARCHAR(36) NOT NULL,
    skill_type ENUM('forge_armes', 'forge_armures', 'alchimie', 'cuisine', 'joaillerie') NOT NULL,
    niveau INT DEFAULT 1,
    experience BIGINT DEFAULT 0,
    specialisation VARCHAR(50),
    PRIMARY KEY (joueur_uuid, skill_type)
);

CREATE TABLE craft_recipes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nom VARCHAR(100) NOT NULL,
    skill_type VARCHAR(20) NOT NULL,
    niveau_requis INT NOT NULL,
    ingredients JSON NOT NULL,
    resultat_item TEXT NOT NULL,
    difficulte INT DEFAULT 50
);

CREATE TABLE craft_history (
    id INT PRIMARY KEY AUTO_INCREMENT,
    joueur_uuid VARCHAR(36) NOT NULL,
    recipe_id INT NOT NULL,
    qualite ENUM('defaillant', 'basique', 'bien_fait', 'excellent', 'parfait', 'legendaire'),
    score_minijeu INT NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 4.1.2 Classes principales
```java
public class CraftingSkill {
    private SkillType type;
    private int level;
    private long experience;
    private String specialization;
    
    public void addExperience(int amount) { /* ... */ }
    public boolean canCraftRecipe(Recipe recipe) { /* ... */ }
}

public class CraftingSession {
    private Player player;
    private Recipe recipe;
    private CraftingMinigame minigame;
    private List<ItemStack> materials;
    private CraftingStation station;
}
```

---

## 5. Interface utilisateur

### 5.1 Interfaces de craft

#### 5.1.1 Menu principal de craft
- **Sélection de compétence** : Onglets par type d'artisanat
- **Liste des recettes** : Filtrées par niveau et matériaux
- **Aperçu du résultat** : Stats et apparence prévisionnelles
- **Inventaire de matériaux** : Gestion des composants

#### 5.1.2 Interface de mini-jeu
- **Barre de progression** : Avancement du craft
- **Indicateurs de qualité** : Score en temps réel
- **Instructions** : Guidance pour les actions
- **Timer** : Temps limité pour certaines actions

### 5.2 Commandes de craft
- `/craft ouvrir` - Ouvrir l'interface de craft
- `/craft recettes <compétence>` - Voir les recettes disponibles
- `/craft compétences` - Afficher les niveaux de compétences
- `/craft station <type>` - Créer une station de craft
- `/craft qualité <objet>` - Voir la qualité d'un objet

### 5.3 Stations de craft spécialisées
- **Forge** : Pour armes et armures métalliques
- **Laboratoire d'alchimie** : Potions et cristaux
- **Cuisine** : Plats et consommables
- **Atelier de joaillerie** : Accessoires et gemmes
- **Établi avancé** : Craft général amélioré

---

## 6. Mécaniques avancées

### 6.1 Système d'amélioration

#### 6.1.1 Renforcement d'équipement
- **Niveaux de renforcement** : +1 à +15 comme dans SAO
- **Matériaux de renforcement** : Pierres et cristaux spéciaux
- **Risque d'échec** : Chance de destruction à hauts niveaux
- **Bonus progressifs** : Stats croissantes par niveau

#### 6.1.2 Enchantements spéciaux
- **Propriétés élémentaires** : Feu, glace, foudre, terre
- **Effets spéciaux** : Drain de vie, critique, vitesse
- **Combinaisons** : Synergie entre enchantements
- **Durabilité** : Usure des enchantements

### 6.2 Économie de l'artisanat
- **Commandes de joueurs** : Système de commandes personnalisées
- **Marché des matériaux** : Fluctuation des prix
- **Réputation de crafter** : Notoriété et bonus clients
- **Boutiques de craft** : Magasins tenus par les joueurs

---

## 7. Recettes et progression

### 7.1 Exemples de recettes

#### 7.1.1 Épée basique (Niveau 1-50)
```yaml
recette_epee_fer:
  nom: "Épée en fer"
  niveau_requis: 15
  ingredients:
    - type: "fer_lingot"
      quantite: 3
    - type: "bois_chene"
      quantite: 1
  difficulte: 30
  temps_craft: 45
```

#### 7.1.2 Potion de soin (Niveau 25-100)
```yaml
recette_potion_soin:
  nom: "Potion de soin mineur"
  niveau_requis: 25
  ingredients:
    - type: "herbe_medicinale"
      quantite: 2
    - type: "eau_pure"
      quantite: 1
  difficulte: 40
  temps_craft: 30
```

### 7.2 Arbre de progression
- **Niveaux 1-100** : Recettes basiques, matériaux communs
- **Niveaux 101-300** : Objets intermédiaires, premiers bonus
- **Niveaux 301-600** : Équipements avancés, matériaux rares
- **Niveaux 601-900** : Objets d'élite, crafts légendaires
- **Niveaux 901-1000** : Artefacts uniques, maître artisan

---

## 8. Intégrations et compatibilité

### 8.1 Plugins compatibles
- **Vault** : Économie pour achat de matériaux
- **ItemsAdder** : Objets et matériaux personnalisés
- **MythicMobs** : Drops de matériaux rares
- **WorldGuard** : Protection des stations de craft

### 8.2 API pour développeurs
```java
public interface CraftingAPI {
    void registerRecipe(Recipe recipe);
    void startCraftingSession(Player player, Recipe recipe);
    void addCustomMaterial(Material material);
    int getPlayerSkillLevel(Player player, SkillType skill);
}
```

---

## 9. Critères d'évaluation

### 9.1 Fonctionnalités obligatoires
- [ ] Système de compétences fonctionnel
- [ ] Mini-jeux de craft implémentés
- [ ] Système de qualité opérationnel
- [ ] Interface utilisateur complète
- [ ] Recettes de base créées

### 9.2 Fonctionnalités bonus
- [ ] Stations de craft spécialisées
- [ ] Système d'amélioration avancé
- [ ] Économie de l'artisanat
- [ ] Effets visuels et sonores
- [ ] API pour autres plugins

### 9.3 Critères techniques
- [ ] Performance optimisée (mini-jeux fluides)
- [ ] Persistance des données fiable
- [ ] Interface responsive et intuitive
- [ ] Équilibrage des recettes
- [ ] Code modulaire et extensible

---

## 10. Planning de développement

| Phase | Durée | Description |
|-------|-------|-------------|
| Architecture système | 2 jours | BDD, classes principales |
| Système de compétences | 3 jours | Progression et expérience |
| Mini-jeux de craft | 5 jours | Mécaniques interactives |
| Système de qualité | 2 jours | Calculs et bonus |
| Interface utilisateur | 4 jours | GUI et stations de craft |
| Recettes et équilibrage | 2 jours | Contenu et balance |
| Tests et optimisation | 2 jours | Performance et bugs |

**Total estimé : 20 jours**

---

*Cahier des charges créé pour le recrutement développeur SAO France - 18 juin 2025*
