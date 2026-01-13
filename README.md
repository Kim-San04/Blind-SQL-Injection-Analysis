<img width="2668" height="638" alt="image" src="https://github.com/user-attachments/assets/36f01261-075a-459b-8519-72055decc443" />

# 🛡️ Étude de la Vulnérabilité : Injection SQL Aveugle (Blind SQL Injection)

## 👥 Équipe de Réalisation
Ce projet a été réalisé par : **Fady Axel BAMBA, Maixente Yakeliomi KOHIO, Pierre Marie KONATE, Cheick Abdel Hadime Hakim SAWADOGO et Mouhamed SALANE**.

---

## 📖 1. Introduction
L'**injection SQL (SQL Injection)** est une vulnérabilité de sécurité web critique qui permet à un attaquant d'insérer des requêtes SQL malveillantes dans les champs d'entrée d'une application. Cette manipulation vise la base de données sous-jacente afin d'accéder, modifier ou supprimer des informations sensibles.

L’**injection SQL aveugle (Blind SQL Injection)** est une variante spécifique où l’attaquant ne reçoit aucun retour d’erreur ou de réponse visible directe de la base de données. L’application répond alors de manière détournée, soit de façon **binaire (vrai ou faux)**, soit par un **délai dans l’exécution** de la requête. Cette technique est particulièrement redoutable lorsque les messages d'erreur SQL explicites sont désactivés ou filtrés.

### 🔄 Comparaison des types d'injections
| Critère | Injection SQL Classique | Injection SQL Aveugle |
| :--- | :--- | :--- |
| **Réponse DB** | Messages d'erreurs et résultats exploitables | Aucune réponse directe de la base |
| **Facilité** | Facile à détecter et à exploiter | Plus difficile, nécessite des techniques spécifiques |
| **Méthodes** | Erreurs SQL, affichage direct des données | Réponses booléennes, délais d'exécution |

---

## ⚙️ 2. Fonctionnement de l'Attaque
L'attaque repose sur l'observation des changements de comportement de l'application face à des entrées malveillantes.

### 2.1 Boolean-Based (Basée sur les booléens)
L'attaquant injecte une requête qui force une réponse conditionnelle (vrai ou faux).
*   **Test de validité :** `id=1' AND 1=1 --` (Si la page se charge normalement, la requête est valide).
*   **Test de vulnérabilité :** `id=1' AND 1=2 --` (Si la page change de comportement ou affiche une erreur générique, l'application est vulnérable).

### 2.2 Time-Based (Basée sur le temps)
L'attaquant force la base de données à attendre un certain temps avant de répondre.
*   **Exemple :** `id=1' AND SLEEP(5) --`.
*   **Interprétation :** Si la page met exactement 5 secondes de plus à répondre, la vulnérabilité est confirmée.

---

## 🛠️ 3. Exploitation et Exfiltration
L'attaquant identifie la faille en envoyant des caractères spéciaux (`'`, `"`, `--`, `#`) pour repérer des erreurs masquées. Une fois confirmée, il peut extraire des données caractère par caractère.

### 📂 Identification de la structure
*   **Détection d'une table :** `id=1' AND (SELECT COUNT(*) FROM information_schema.tables WHERE table_name='utilisateurs') > 0 --`.
*   **Longueur d'une colonne :** `id=1' AND (SELECT LENGTH(column_name) FROM information_schema.columns WHERE table_name='utilisateurs' LIMIT 1) > 5 --`.

### 🔓 Exfiltration de données
Pour récupérer un contenu (comme un nom d'utilisateur), l'attaquant utilise des tests ASCII :
*   **Exemple :** `id=1' AND ASCII(SUBSTRING((SELECT username FROM utilisateurs LIMIT 1),1,1))=97 --`.
*   Si la page répond normalement (Vrai), le premier caractère est 'a' (code ASCII 97).

---

## 📊 4. Étude de Cas : Niveaux de Sécurité (DVWA)
L'analyse de la plateforme **DVWA** montre l'évolution des mesures de protection :

*   **Niveau LOW :** L'entrée `id` est récupérée via `$_GET` et injectée sans aucune validation ni filtrage.
  <img width="940" height="283" alt="image" src="https://github.com/user-attachments/assets/21158ac4-316e-4fbf-9a3e-bea44f5567ce" />

---
*   **Niveau MEDIUM :** Utilisation de `mysql_real_escape_string()`, mais l'absence de guillemets autour de l'ID dans la requête SQL permet de contourner l'échappement.
  <img width="940" height="267" alt="image" src="https://github.com/user-attachments/assets/97b252be-24d8-462b-9af5-ba00c89257fa" />

---
*   **Niveau HIGH :** L'ID est passé par un cookie (`$_COOKIE`), ce qui oblige l'usage d'outils comme **Burp Suite** pour manipuler les requêtes côté client.
  <img width="939" height="275" alt="image" src="https://github.com/user-attachments/assets/f14278a1-5ae9-4719-a08c-e346efb92535" />

---
*   **Niveau IMPOSSIBLE :** Sécurité maximale via un typage strict (`intval()`), des **requêtes préparées (PDO)**, des tokens anti-CSRF et une gestion d'erreur opaque.
  <img width="939" height="510" alt="image" src="https://github.com/user-attachments/assets/00b52b55-9f24-4948-b426-32ba6d0f3ca2" />

---

## 🛡️ 5. Méthodes de Protection et Prévention
Une protection efficace repose sur plusieurs piliers fondamentaux :

1.  **Requêtes Préparées (Prepared Statements) :** C'est la défense la plus efficace car elle sépare les données de la structure de la commande SQL.
    *   *Exemple PHP/PDO :* `$stmt = $pdo->prepare("SELECT * FROM utilisateurs WHERE id = :id"); $stmt->execute(['id' => $id]);`.
2.  **Validation et Filtrage :** Vérifier que les entrées correspondent au format attendu (ex: uniquement des nombres pour un ID) et échapper les caractères dangereux.
3.  **Principe du moindre privilège :** Attribuer uniquement les permissions nécessaires aux comptes SQL et désactiver les fonctions dangereuses comme `LOAD_FILE()` ou `xp_cmdshell`.
4.  **Web Application Firewall (WAF) :** Détecter et bloquer les patterns d'attaque connus avant qu'ils n'atteignent l'application.

---

## ⚠️ 6. Conséquences et Risques
Une exploitation réussie peut avoir des conséquences désastreuses :
*   **Accès illimité** aux données sans authentification.
*   **Vol d'informations sensibles** (identifiants, emails, données personnelles).
*   **Contrôle total du système** via l'exécution de commandes système.
*   **Coûts élevés** de réparation et dommages irréparables à la réputation de l'entreprise.

---

## 🏁 Conclusion
L'injection SQL aveugle est une attaque redoutable car elle peut rester active longtemps sans être détectée. Une défense robuste nécessite une approche multicouche combinant code sécurisé, filtrage rigoureux et restrictions des privilèges en base de données.
