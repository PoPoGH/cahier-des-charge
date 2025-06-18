# Cahier des charges - Système de téléportation entre villes
## Serveur Minecraft SAO France - Version Simplifiée

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans SAO, les joueurs peuvent se téléporter entre les villes qu'ils ont déjà visitées. Ce système doit permettre aux joueurs de voyager rapidement tout en gardant une logique de progression.

### 1.2 Objectifs
- Permettre la téléportation entre villes découvertes
- Système de coût en monnaie du serveur
- Interface simple et intuitive
- Empêcher les abus et exploits

---

## 2. Fonctionnalités demandées

### 2.1 Fonctionnalités principales
- **Découverte de villes** : Le joueur doit visiter une ville pour la débloquer
- **Menu de téléportation** : Interface GUI pour sélectionner la destination
- **Coût de téléportation** : Prix selon la distance (10 cols par 100 blocs)
- **Cooldown** : 30 secondes entre chaque téléportation

### 2.2 Villes à configurer
- **Ville de départ** : Toujours disponible (gratuite)
- **Ville niveau 10** : Déblocage automatique au niveau 10
- **Ville niveau 25** : Déblocage automatique au niveau 25
- **Villes découvertes** : En visitant physiquement la zone

---

## 3. Spécifications techniques

### 3.1 Base de données simple
```sql
CREATE TABLE player_teleports (
    player_uuid VARCHAR(36) NOT NULL,
    ville_name VARCHAR(50) NOT NULL,
    discovered_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (player_uuid, ville_name)
);
```

### 3.2 Configuration des villes
```yaml
# config.yml
villes:
  spawn:
    name: "Ville de départ"
    world: "world"
    x: 0
    y: 64
    z: 0
    cost: 0
    level_required: 0
    always_unlocked: true
  
  ville_niveau10:
    name: "Première cité"
    world: "world"
    x: 1000
    y: 64
    z: 500
    cost: 50
    level_required: 10
    always_unlocked: false
```

---

## 4. Interface utilisateur

### 4.1 Commandes
- `/tp` ou `/teleport` - Ouvrir le menu de téléportation
- `/tp <ville>` - Téléportation directe vers une ville
- `/tp list` - Liste des villes disponibles

### 4.2 Menu GUI
- Interface avec les villes sous forme d'items
- Couleur verte = disponible
- Couleur rouge = non débloquée
- Couleur jaune = pas assez d'argent

---

## 5. Règles de gestion

### 5.1 Conditions de téléportation
- Le joueur doit avoir découvert la ville
- Avoir assez d'argent pour payer
- Ne pas être en combat (5 secondes)
- Respecter le cooldown

### 5.2 Découverte de villes
- Zone de détection : rayon de 50 blocs autour du centre ville
- Message de confirmation : "Vous avez découvert [Nom de la ville] !"
- Sauvegarde automatique dans la base de données

---

## 6. Critères d'évaluation

### 6.1 Fonctionnalités obligatoires (80 points)
- [ ] Système de découverte de villes (20 points)
- [ ] Interface GUI fonctionnelle (20 points)
- [ ] Calcul des coûts correct (15 points)
- [ ] Sauvegarde des données (15 points)
- [ ] Gestion des erreurs (10 points)

### 6.2 Fonctionnalités bonus (20 points)
- [ ] Effets visuels de téléportation (5 points)
- [ ] Sons d'interface (5 points)
- [ ] Configuration via fichier (5 points)
- [ ] Commandes admin pour gérer les villes (5 points)

---

## 7. Livrables attendus

### 7.1 Code source
- Plugin Java complet et compilable
- Fichier de configuration d'exemple
- README avec instructions d'installation

### 7.2 Test
- Démonstration sur serveur de test
- Au moins 3 villes configurées
- Test avec différents niveaux de joueurs

---

## 8. Aide et ressources

### 8.1 APIs utiles
- **Bukkit Location** : Pour les coordonnées
- **Vault Economy** : Pour la gestion de l'argent
- **Inventory GUI** : Pour l'interface

### 8.2 Exemple de code de base
```java
@EventHandler
public void onPlayerMove(PlayerMoveEvent event) {
    Player player = event.getPlayer();
    // Vérifier si le joueur est près d'une ville non découverte
    // Si oui, débloquer la ville
}
```

---

## 9. Planning suggéré

| Jour | Tâche |
|------|-------|
| 1 | Configuration et structure du plugin |
| 2 | Système de découverte de villes |
| 3 | Interface GUI et téléportation |
| 4 | Tests et corrections |

**Total estimé : 4 jours**

---

*Version simplifiée pour candidats débutants - SAO France*
