# Cahier des charges - Système de boutique personnelle
## Serveur Minecraft SAO France - Version Simplifiée

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans l'univers SAO, les joueurs peuvent ouvrir des boutiques pour vendre leurs objets à d'autres joueurs. Ce système doit permettre un commerce simple entre joueurs.

### 1.2 Objectifs
- Permettre aux joueurs de créer une boutique personnelle
- Vendre des objets même en étant hors ligne
- Interface simple d'achat pour les clients
- Gestion automatique des transactions

---

## 2. Fonctionnalités demandées

### 2.1 Fonctionnalités principales
- **Création de boutique** : `/shop create` dans sa région protégée
- **Ajout d'objets** : `/shop add <prix>` avec l'item en main
- **Interface client** : Clic droit sur un panneau pour ouvrir la boutique
- **Historique des ventes** : Voir ses ventes récentes

### 2.2 Types d'objets vendables
- Tous les objets normaux de Minecraft
- Objets customisés du serveur SAO
- Limitation : pas d'objets interdits (épées en diamant par exemple)

---

## 3. Interface utilisateur

### 3.1 Pour le vendeur
```
/shop create         - Créer sa boutique ici
/shop add <prix>     - Ajouter l'item en main à la vente
/shop remove <slot>  - Retirer un item de la vente
/shop list          - Voir ses objets en vente
/shop earnings      - Voir ses gains
```

### 3.2 Pour l'acheteur
```
Clic droit sur panneau "BOUTIQUE" → Ouvre l'interface de la boutique
Interface GUI avec :
- Items en vente avec prix
- Clic gauche = acheter 1
- Clic droit = acheter en quantité
- Indication si pas assez d'argent
```

---

## 4. Système de boutique

### 4.1 Installation d'une boutique
1. Joueur tape `/shop create` dans sa région
2. Un panneau apparaît avec le nom du joueur
3. Message : "Boutique créée ! Ajoutez des objets avec /shop add <prix>"

### 4.2 Fonctionnement des ventes
```
Client achète un item:
1. Vérifier qu'il a assez d'argent
2. Retirer l'argent du client
3. Ajouter l'argent au vendeur (même hors ligne)
4. Donner l'item au client
5. Sauvegarder la transaction
```

---

## 5. Base de données

### 5.1 Table boutiques
```sql
CREATE TABLE boutiques (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proprietaire VARCHAR(36) NOT NULL,
    monde VARCHAR(50) NOT NULL,
    x INT NOT NULL,
    y INT NOT NULL,
    z INT NOT NULL,
    date_creation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    active BOOLEAN DEFAULT TRUE
);
```

### 5.2 Table items en vente
```sql
CREATE TABLE boutique_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    boutique_id INT NOT NULL,
    item_data TEXT NOT NULL,
    prix DECIMAL(10,2) NOT NULL,
    quantite INT NOT NULL,
    slot_gui INT NOT NULL,
    FOREIGN KEY (boutique_id) REFERENCES boutiques(id)
);
```

### 5.3 Table transactions
```sql
CREATE TABLE transactions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    boutique_id INT NOT NULL,
    acheteur VARCHAR(36) NOT NULL,
    item_name VARCHAR(100) NOT NULL,
    quantite INT NOT NULL,
    prix_total DECIMAL(10,2) NOT NULL,
    date_achat TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 6. Règles de gestion

### 6.1 Limitations
- Une seule boutique par joueur
- Maximum 27 types d'objets différents (taille d'un inventaire)
- Prix minimum : 1 col, maximum : 1 000 000 cols
- Boutique supprimée si propriétaire inactif 30 jours

### 6.2 Sécurité
- Seul le propriétaire peut modifier sa boutique
- Vérification des permissions de construction dans la zone
- Protection contre les achats en spam

---

## 7. Interface GUI

### 7.1 Interface vendeur (gestion)
```
┌─────────────────────────────────┐
│     Ma Boutique - Gestion       │
├─────────────────────────────────┤
│ [Item1] [Item2] [Item3] [...] │ Objets en vente
│ [Item4] [Item5] [Item6] [...] │
│ [Item7] [Item8] [Item9] [...] │
├─────────────────────────────────┤
│ [Ajouter] [Retirer] [Gains]     │ Boutons d'action
└─────────────────────────────────┘
```

### 7.2 Interface client (achat)
```
┌─────────────────────────────────┐
│   Boutique de [Pseudo]          │
├─────────────────────────────────┤
│ [Épée] [Arc] [Pomme] [...]     │ Items disponibles
│ 100c   50c   5c                 │ Prix affichés
│                                 │
│ Clic gauche = Acheter           │ Instructions
│ Votre argent: 1,250 cols        │ Solde du joueur
└─────────────────────────────────┘
```

---

## 8. Exemple de code

### 8.1 Création de boutique
```java
public boolean creerBoutique(Player player, Location location) {
    // Vérifier que le joueur n'a pas déjà une boutique
    if (aBoutiqueExistante(player)) {
        player.sendMessage("Vous avez déjà une boutique !");
        return false;
    }
    
    // Créer le panneau
    Block block = location.getBlock();
    block.setType(Material.OAK_SIGN);
    Sign sign = (Sign) block.getState();
    sign.setLine(0, "BOUTIQUE");
    sign.setLine(1, player.getName());
    sign.update();
    
    // Sauvegarder en base
    sauvegarderBoutique(player, location);
    
    player.sendMessage("Boutique créée ! Utilisez /shop add <prix> pour ajouter des objets.");
    return true;
}
```

### 8.2 Achat d'item
```java
public void acheterItem(Player acheteur, int boutiqueId, int slot) {
    BoutiqueItem item = getItemBoutique(boutiqueId, slot);
    
    if (item == null) {
        acheteur.sendMessage("Cet objet n'est plus disponible !");
        return;
    }
    
    double prix = item.getPrix();
    if (!economy.has(acheteur, prix)) {
        acheteur.sendMessage("Vous n'avez pas assez d'argent ! (Requis: " + prix + " cols)");
        return;
    }
    
    // Transaction
    economy.withdrawPlayer(acheteur, prix);
    economy.depositPlayer(Bukkit.getOfflinePlayer(item.getProprietaire()), prix);
    
    // Donner l'item
    acheteur.getInventory().addItem(item.getItemStack());
    
    // Enregistrer la vente
    enregistrerTransaction(boutiqueId, acheteur, item);
    
    acheteur.sendMessage("Achat effectué ! Vous avez acheté " + item.getNom() + " pour " + prix + " cols.");
}
```

---

## 9. Critères d'évaluation

### 9.1 Fonctionnalités obligatoires (80 points)
- [ ] Création de boutique fonctionnelle (20 points)
- [ ] Ajout/retrait d'objets en vente (20 points)
- [ ] Interface d'achat pour clients (20 points)
- [ ] Gestion des transactions économiques (20 points)

### 9.2 Fonctionnalités bonus (20 points)
- [ ] Interface GUI soignée (8 points)
- [ ] Historique des ventes détaillé (4 points)
- [ ] Système de recherche de boutiques (4 points)
- [ ] Effets visuels/sonores (4 points)

---

## 10. Tests à effectuer

### 10.1 Tests de base
- Créer une boutique
- Ajouter des objets à vendre
- Acheter depuis une autre boutique
- Vérifier la réception de l'argent

### 10.2 Tests avancés
- Tester avec un vendeur hors ligne
- Vérifier les limites (prix, quantité)
- Tester la suppression d'objets
- Vérifier la persistance après redémarrage

---

## 11. Livrables

### 11.1 Code source
- Plugin complet et fonctionnel
- Configuration Vault Economy
- Documentation d'installation

### 11.2 Démonstration
- Vidéo de 5 minutes montrant :
  - Création d'une boutique
  - Ajout d'objets en vente
  - Achat depuis la boutique
  - Gestion des gains

---

## 12. Planning suggéré

| Jour | Tâche |
|------|-------|
| 1-2 | Structure du plugin et base de données |
| 3 | Création et gestion des boutiques |
| 4 | Interface d'achat client |
| 5 | Tests et interface GUI |
| 6 | Finitions et bonus |

**Total estimé : 5-6 jours**

---

*Version simplifiée pour candidats débutants/intermédiaires - SAO France*
