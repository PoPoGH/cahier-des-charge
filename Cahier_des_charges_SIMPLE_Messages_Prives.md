# Cahier des charges - Système de messages privés SAO
## Serveur Minecraft SAO France - Version Simplifiée

---

## 1. Contexte et objectifs

### 1.1 Contexte
Dans SAO, les joueurs peuvent s'envoyer des messages privés même quand ils ne sont pas connectés en même temps. Ce système doit reproduire cette fonctionnalité de manière simple.

### 1.2 Objectifs
- Permettre d'envoyer des messages à des joueurs hors ligne
- Stocker les messages non lus
- Interface simple pour consulter ses messages
- Notifications quand on reçoit un nouveau message

---

## 2. Fonctionnalités demandées

### 2.1 Fonctionnalités principales
- **Envoi de messages** : `/msg <joueur> <message>`
- **Consultation des messages** : `/messages` ou `/msg list`
- **Messages non lus** : Compteur de nouveaux messages
- **Historique** : Garder les 50 derniers messages reçus

### 2.2 Notifications
- Message à la connexion si nouveaux messages
- Son de notification lors de la réception
- Indication visuelle dans le chat

---

## 3. Base de données

### 3.1 Table messages
```sql
CREATE TABLE messages_prives (
    id INT PRIMARY KEY AUTO_INCREMENT,
    expediteur VARCHAR(36) NOT NULL,
    destinataire VARCHAR(36) NOT NULL,
    message TEXT NOT NULL,
    date_envoi TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    lu BOOLEAN DEFAULT FALSE
);
```

### 3.2 Nettoyage automatique
- Supprimer les messages de plus de 30 jours
- Garder maximum 50 messages par joueur

---

## 4. Interface utilisateur

### 4.1 Commandes principales
- `/msg <joueur> <message>` - Envoyer un message
- `/messages` - Voir tous ses messages
- `/messages <joueur>` - Conversation avec un joueur spécifique
- `/msg clear` - Effacer tous ses messages

### 4.2 Interface GUI (bonus)
- Menu avec liste des conversations
- Clic pour ouvrir une conversation
- Bouton pour marquer comme lu

---

## 5. Fonctionnement détaillé

### 5.1 Envoi de message
```
Joueur tape: /msg Steve Salut comment ça va ?

Si Steve est connecté:
- Steve reçoit : [MP] Alice: Salut comment ça va ?
- Alice reçoit : [MP -> Steve] Salut comment ça va ?

Si Steve est hors ligne:
- Message sauvegardé dans la BDD
- Alice reçoit : Message envoyé à Steve (hors ligne)
```

### 5.2 Réception de message
```
Steve se connecte:
- Message: Vous avez 3 nouveaux messages privés
- Son de notification
- Tapez /messages pour les consulter
```

---

## 6. Règles de gestion

### 6.1 Limitations
- Maximum 10 messages par minute par joueur
- Messages de 500 caractères maximum
- Pas de messages vers soi-même
- Bloquer les joueurs bannis

### 6.2 Modération
- Log de tous les messages dans un fichier
- Commande admin pour voir les messages d'un joueur
- Possibilité de bloquer certains mots

---

## 7. Exemple de code

### 7.1 Structure de base
```java
public class MessagePrivePlugin extends JavaPlugin {
    
    @Override
    public void onEnable() {
        // Initialiser la base de données
        // Enregistrer les commandes
        // Enregistrer les événements
    }
    
    public void envoyerMessage(Player expediteur, String destinataire, String message) {
        // Vérifier si le destinataire existe
        // Sauvegarder en base
        // Notifier si connecté
    }
}
```

### 7.2 Commande exemple
```java
@Override
public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
    if (!(sender instanceof Player)) return false;
    
    Player player = (Player) sender;
    
    if (args.length < 2) {
        player.sendMessage("Usage: /msg <joueur> <message>");
        return true;
    }
    
    String destinataire = args[0];
    String message = String.join(" ", Arrays.copyOfRange(args, 1, args.length));
    
    envoyerMessage(player, destinataire, message);
    return true;
}
```

---

## 8. Critères d'évaluation

### 8.1 Fonctionnalités obligatoires (80 points)
- [ ] Envoi de messages fonctionnel (25 points)
- [ ] Sauvegarde en base de données (20 points)
- [ ] Consultation des messages (20 points)
- [ ] Notifications de base (15 points)

### 8.2 Fonctionnalités bonus (20 points)
- [ ] Interface GUI (8 points)
- [ ] Sons et effets (4 points)
- [ ] Commandes admin (4 points)
- [ ] Nettoyage automatique (4 points)

---

## 9. Tests à effectuer

### 9.1 Tests de base
- Envoyer un message à un joueur connecté
- Envoyer un message à un joueur hors ligne
- Se reconnecter et voir les nouveaux messages
- Consulter l'historique des messages

### 9.2 Tests avancés
- Envoyer 50+ messages et vérifier la limite
- Tester avec des caractères spéciaux
- Vérifier le nettoyage automatique
- Tester les notifications

---

## 10. Livrables

### 10.1 Code source
- Plugin compilable avec toutes les fonctionnalités
- Configuration de base de données
- Documentation basique

### 10.2 Démonstration
- Vidéo de 3-5 minutes montrant :
  - Envoi de messages
  - Consultation des messages
  - Notifications
  - Interface (si implémentée)

---

## 11. Planning suggéré

| Jour | Tâche |
|------|-------|
| 1 | Structure du plugin et base de données |
| 2 | Commandes d'envoi et réception |
| 3 | Interface de consultation |
| 4 | Notifications et tests |
| 5 | Interface GUI (bonus) et finitions |

**Total estimé : 4-5 jours**

---

## 12. Ressources utiles

### 12.1 Documentation
- [Bukkit CommandExecutor](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/command/CommandExecutor.html)
- [Bukkit OfflinePlayer](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/OfflinePlayer.html)
- [Tutoriel JDBC MySQL](https://www.w3schools.com/java/java_mysql.asp)

### 12.2 Exemples de plugins similaires
- EssentialsX Mail (pour inspiration)
- PrivateMessage plugins sur SpigotMC

---

*Version simplifiée pour candidats débutants - SAO France*
