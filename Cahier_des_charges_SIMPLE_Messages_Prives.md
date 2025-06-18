# Cahier des charges - Système de lettres et courrier SAO
## Serveur Minecraft SAO France - Version Simplifiée

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans l'univers médiéval-fantastique de SAO, les joueurs communiquent par lettres et courriers, comme dans un vrai MMORPG. Le système utilise des objets physiques (papier, plume) pour créer une expérience immersive sans commandes.

### 1.2 Objectifs
- Système de courrier immersif avec objets physiques
- Écriture de lettres sur papier avec livre et plume
- Envoi via des boîtes aux lettres dans les villes
- Réception automatique dans sa boîte personnelle

---

## 2. Fonctionnalités demandées

### 2.1 Système
- **Commande unique** : `/lettre <joueur> <message>` - Simple et efficace
- **Livraison automatique** : Lettre apparaît dans l'inventaire du destinataire
- **Objet physique** : Vraie lettre (livre signé) avec le message
- **Coût d'envoi** : 10 cols pour l'envoi (encre et papier)

### 2.2 Réception immersive
- **Item physique** : "Lettre de [expéditeur]" dans l'inventaire
- **Lecture** : Clic droit pour lire le contenu
- **Conservation** : Le joueur garde la lettre ou peut la jeter
- **Notification** : Message à la connexion si nouvelles lettres

### 2.3 Avantages
- **Une seule commande** à retenir
- **Objet réel** que le joueur possède
- **Immersion** sans complexité
- **Coût économique** intégré

---

## 3. Base de données

### 3.1 Table lettres
```sql
CREATE TABLE lettres (
    id INT PRIMARY KEY AUTO_INCREMENT,
    expediteur VARCHAR(36) NOT NULL,
    destinataire VARCHAR(36) NOT NULL,
    message TEXT NOT NULL,
    date_envoi TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    livree BOOLEAN DEFAULT FALSE
);
```

### 3.2 Nettoyage automatique
- Supprimer les lettres livrées de plus de 7 jours
- Supprimer les lettres non livrées de plus de 30 jours

---

## 4. Interface utilisateur

### 4.1 Une seule commande !
```
/lettre <joueur> <message>

Exemples:
/lettre Steve Salut ! Comment ça va ?
/lettre Alice Merci pour ton aide hier !
/lettre Bob RDV ce soir à 20h à la taverne
```

### 4.2 Réception automatique
```
Quand le destinataire se connecte:
1. Message: "Vous avez reçu 2 nouvelles lettres !"
2. Les lettres apparaissent automatiquement dans son inventaire
3. Items: "Lettre de Steve", "Lettre d'Alice"
4. Clic droit sur une lettre → Lecture du message
```

### 4.3 Lecture d'une lettre
```
Clic droit sur "Lettre de Steve" → Ouvre un livre avec:

┌─────────────────────────────────┐
│         Lettre de Steve         │
├─────────────────────────────────┤
│                                 │
│ Salut ! Comment ça va ?         │
│                                 │
│ J'espère que tu vas bien.       │
│ On se voit bientôt !            │
│                                 │
│                                 │
│         - Steve                 │
│   18 juin 2025, 14:30          │
└─────────────────────────────────┘
```

---

## 5. Fonctionnement détaillé

### 5.1 Cycle d'une lettre
```
1. ENVOI (Alice)
   - Tape: /lettre Steve Salut comment ça va ?
   - Vérification: Alice a-t-elle 10 cols ?
   - Déduction: 10 cols retirés du compte d'Alice
   - Message: "Lettre envoyée à Steve pour 10 cols"
   - Sauvegarde en base de données

2. RÉCEPTION (Steve - à sa prochaine connexion)
   - Steve se connecte
   - Vérification: A-t-il des lettres non livrées ?
   - Création: Item "Lettre d'Alice" ajouté à son inventaire
   - Notification: "Vous avez reçu 1 nouvelle lettre !"
   - Marquage: Lettre marquée comme livrée en BDD

3. LECTURE (Steve)
   - Clic droit sur "Lettre d'Alice"
   - Ouverture: Livre avec le message formaté
   - Contenu: Message + signature + date
```

### 5.2 Avantages de cette approche
- **Simple** : Une seule commande à retenir
- **Immersif** : Objets physiques dans l'inventaire
- **Pratique** : Pas de déplacement vers des boîtes
- **Économique** : Coût raisonnable de 10 cols
- **Persistant** : Les lettres restent dans l'inventaire

---

## 6. Règles de gestion

### 6.1 Règles simples
- **Coût fixe** : 10 cols par lettre (économique)
- **Taille limite** : 200 caractères maximum (lisible)
- **Délai de livraison** : Immédiat à la prochaine connexion
- **Conservation** : Lettres dans l'inventaire jusqu'à destruction

### 6.2 Limitations pratiques
- **Anti-spam** : Maximum 5 lettres par minute par joueur
- **Destinataire valide** : Le joueur doit exister sur le serveur
- **Argent requis** : Vérification du solde avant envoi
- **Inventaire plein** : Lettre envoyée au sol si inventaire plein

### 6.3 Gestion automatique
- **Lettres perdues** : Si inventaire plein, lettre droppée au sol
- **Nettoyage BDD** : Suppression des lettres livrées après 7 jours
- **Modération** : Log de toutes les lettres pour les admins

---


## 7. Critères d'évaluation

### 7.1 Fonctionnalités obligatoires
- [ ] Commande /lettre fonctionnelle
- [ ] Vérification économique (coût 10 cols)
- [ ] Création d'items "lettres" dans l'inventaire
- [ ] Livraison à la connexion
- [ ] Sauvegarde en base de données

### 7.2 Fonctionnalités bonus
- [ ] Interface de lecture soignée (livre formaté)
- [ ] Gestion inventaire plein (drop au sol)
- [ ] Anti-spam (limite 5 lettres/minute)
- [ ] Commandes admin pour modération

### 7.3 Simplicité d'usage
- [ ] Une seule commande à retenir
- [ ] Pas de craft complexe
- [ ] Pas de déplacement requis
- [ ] Réception automatique

---

## 8. Tests à effectuer

### 8.1 Tests de base
- Envoyer une lettre avec `/lettre Steve Salut !`
- Vérifier la déduction de 10 cols
- Se connecter avec le compte du destinataire
- Vérifier la réception de l'item "Lettre de [expéditeur]"
- Clic droit sur la lettre pour lire le contenu

### 8.2 Tests de limites
- Tester sans argent suffisant
- Tester avec message trop long (>200 caractères)
- Tester avec destinataire inexistant
- Tester avec inventaire plein (lettre au sol)
- Tester l'anti-spam (>5 lettres/minute)

### 8.3 Tests de persistance
- Envoyer une lettre à un joueur hors ligne
- Redémarrer le serveur
- Se connecter avec le destinataire
- Vérifier que la lettre est bien livrée

---

## 9 Livrables

### 9.1 Code source
- Plugin complet
- Documentation d'installation et configuration

---