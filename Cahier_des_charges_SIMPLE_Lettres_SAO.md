# Cahier des charges - Système de lettres et courrier SAO
## Serveur Minecraft SAO France

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans l'univers de Sword Art Online, les joueurs peuvent s'envoyer des lettres pour communiquer de manière immersive. Ce système doit créer une expérience de courrier médiéval-fantastique simple et intuitive.

### 1.2 Objectifs
- Créer un système de correspondance immersif
- Permettre aux joueurs de s'envoyer des lettres physiques
- Intégrer un coût économique réaliste
- Offrir une alternative au chat pour les messages importants

---

## 2. Spécifications fonctionnelles

### 2.1 Fonctionnalités principales

#### 2.1.1 Envoi de lettres
- **Commande simple** : `/lettre <destinataire> <message>`
- **Coût économique** : 10 cols par lettre envoyée
- **Vérifications** : Destinataire valide et argent suffisant
- **Limite de taille** : Maximum 200 caractères par message

#### 2.1.2 Réception automatique
- **Livraison à la connexion** : Lettres livrées automatiquement
- **Objet physique** : Item "Lettre de [expéditeur]" dans l'inventaire
- **Lecture immersive** : Clic droit pour ouvrir le livre de la lettre
- **Conservation** : Le joueur peut garder ou détruire ses lettres

#### 2.1.3 Interface utilisateur
- **Notifications** : Alerte à la connexion si nouvelles lettres
- **Gestion automatique** : Inventaire plein = lettre au sol
- **Anti-spam** : Limitation à 5 lettres par minute
- **Modération** : Log de tous les messages pour les admins

### 2.2 Fonctionnalités secondaires
- **Formatage automatique** : Lettres avec signature et date
- **Persistance** : Conservation des lettres non livrées
- **Nettoyage automatique** : Suppression des anciennes lettres

---

## 3. Spécifications techniques

### 3.1 Technologies requises
- **Plugin Minecraft** : Paper 1.21.3
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
   - Commande /lettre     - Validation messages  - Base données
   - Lecture de lettres   - Gestion économie     - Fichiers config
   - Notifications        - Livraison auto       - Logs modération
```

### 3.3 Structures de données

#### 3.3.1 Table Lettres
```sql
CREATE TABLE lettres (
    id INT PRIMARY KEY AUTO_INCREMENT,
    expediteur VARCHAR(36) NOT NULL,
    destinataire VARCHAR(36) NOT NULL,
    message TEXT NOT NULL,
    date_envoi TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    livree BOOLEAN DEFAULT FALSE,
    cout DECIMAL(10,2) DEFAULT 10.00
);
```

#### 3.3.2 Table Statistiques (optionnel)
```sql
CREATE TABLE lettres_stats (
    joueur_uuid VARCHAR(36) PRIMARY KEY,
    lettres_envoyees INT DEFAULT 0,
    lettres_recues INT DEFAULT 0,
    cols_depenses DECIMAL(15,2) DEFAULT 0,
    derniere_lettre TIMESTAMP NULL
);
```

---

## 4. Interface utilisateur

### 4.1 Commandes principales
- `/lettre <destinataire> <message>` - Envoyer une lettre
- `/lettre help` - Aide sur l'utilisation du système

### 4.2 Exemple d'utilisation
```
/lettre Steve Salut ! Comment ça va ?
→ Message: "Lettre envoyée à Steve pour 10 cols !"

/lettre Alice Merci pour ton aide hier soir
→ Message: "Lettre envoyée à Alice pour 10 cols !"

/lettre Bob RDV ce soir 20h taverne du centre
→ Message: "Lettre envoyée à Bob pour 10 cols !"
```

### 4.3 Interface de lecture
```
Réception (à la connexion):
"Vous avez reçu 2 nouvelles lettres !"

Dans l'inventaire:
[📜 Lettre de Steve]
[📜 Lettre d'Alice] 

Clic droit sur une lettre → Ouvre le livre:
┌─────────────────────────────────┐
│         Lettre de Steve         │
├─────────────────────────────────┤
│                                 │
│ Salut ! Comment ça va ?         │
│                                 │
│ J'espère que tu vas bien.       │
│ À bientôt !                     │
│                                 │
│         - Steve                 │
│   18 juin 2025, 14:30          │
└─────────────────────────────────┘
```

### 4.4 Messages système
- **Succès** : "Lettre envoyée à [destinataire] pour 10 cols !"
- **Erreur argent** : "Vous n'avez pas assez d'argent ! (10 cols requis)"
- **Erreur destinataire** : "Le joueur [nom] n'existe pas !"
- **Erreur taille** : "Message trop long ! (200 caractères max)"
- **Anti-spam** : "Vous envoyez trop de lettres ! Attendez un peu."

---

## 5. Règles de gestion

### 5.1 Tarification
- **Coût fixe** : 10 cols par lettre envoyée
- **Déduction automatique** : Vérification du solde avant envoi
- **Pas de remboursement** : Lettres payées même si destinataire absent

### 5.2 Restrictions
- **Taille maximale** : 200 caractères par message
- **Anti-spam** : Maximum 5 lettres par minute par joueur
- **Destinataire valide** : Le joueur doit exister sur le serveur
- **Argent requis** : Vérification obligatoire avant traitement

### 5.3 Permissions
- `lettres.send` - Envoyer des lettres (par défaut)
- `lettres.receive` - Recevoir des lettres (par défaut)
- `lettres.admin` - Voir les logs et statistiques

---

## 6. Critères d'évaluation

### 6.1 Fonctionnalités obligatoires (80 points)
- [ ] Commande `/lettre` fonctionnelle (25 points)
- [ ] Vérification économique (coût 10 cols) (15 points)
- [ ] Création d'items "lettres" dans l'inventaire (20 points)
- [ ] Livraison automatique à la connexion (15 points)
- [ ] Sauvegarde en base de données (5 points)

### 6.2 Fonctionnalités bonus (20 points)
- [ ] Interface de lecture soignée (livre formaté) (8 points)
- [ ] Gestion inventaire plein (drop au sol) (4 points)
- [ ] Anti-spam (limite 5 lettres/minute) (4 points)
- [ ] Commandes admin pour modération (4 points)

### 6.3 Critères techniques
- [ ] Code propre et documenté
- [ ] Gestion des erreurs appropriée
- [ ] Performance optimisée
- [ ] Interface utilisateur intuitive
- [ ] Integration économique fonctionnelle

---

## 7. Livrables attendus

### 7.1 Code source
- Plugin complet avec toutes les fonctionnalités
- Configuration de base de données
- Documentation technique

### 7.2 Documentation
- Guide d'installation
- Manuel utilisateur
- Exemples de configuration

### 7.3 Tests
- Démonstration fonctionnelle
- Tests de cas d'usage
- Validation des performances

---

## 8. Planning suggéré

| Phase | Durée | Description |
|-------|-------|-------------|
| Architecture | 1 jour | Conception technique et base de données |
| Commande de base | 1 jour | Système d'envoi et validation |
| Système de livraison | 1 jour | Réception et création d'items |
| Interface de lecture | 1 jour | Livres formatés et interaction |
| Tests et polish | 1 jour | Tests et corrections |

**Total estimé : 5 jours**

---

## 9. Contact et questions

Pour toute question sur ce cahier des charges, veuillez contacter l'équipe de développement SAO France.

**Bonne chance pour votre candidature !**

---

*Document créé le 18 juin 2025 pour le recrutement développeur SAO France*
