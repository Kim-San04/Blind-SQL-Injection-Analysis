<div align="center">

![Header](https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,100:00f2ff&height=120&section=header&text=&fontSize=0)

</div>

<img width="2668" height="638" alt="image" src="https://github.com/user-attachments/assets/36f01261-075a-459b-8519-72055decc443" />

# ð¡ï¸ Ãtude de la VulnÃ©rabilitÃ© : Injection SQL Aveugle (Blind SQL Injection)

<div align="center">

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) ![SQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white) ![OWASP](https://img.shields.io/badge/OWASP_A03-Critical-red?style=for-the-badge) ![SQLMap](https://img.shields.io/badge/SQLMap-CC0000?style=for-the-badge&logo=linux&logoColor=white) ![Burp Suite](https://img.shields.io/badge/Burp_Suite-FF6633?style=for-the-badge&logo=burpsuite&logoColor=white)

![Efrei](https://img.shields.io/badge/Efrei_Bordeaux-Mastère_Cybersécurité-purple?style=flat-square) ![Type](https://img.shields.io/badge/Type-Web_Security_Research-orange?style=flat-square) ![Status](https://img.shields.io/badge/Status-Terminé-success?style=flat-square)

> ⚠️ **Avertissement légal** : Ce projet est réalisé dans un environnement de laboratoire contrôlé à des fins strictement pédagogiques.

</div>



---

## ð 1. Introduction
L'**injection SQL (SQL Injection)** est une vulnÃ©rabilitÃ© de sÃ©curitÃ© web critique qui permet Ã  un attaquant d'insÃ©rer des requÃªtes SQL malveillantes dans les champs d'entrÃ©e d'une application. Cette manipulation vise la base de donnÃ©es sous-jacente afin d'accÃ©der, modifier ou supprimer des informations sensibles.

Lâ**injection SQL aveugle (Blind SQL Injection)** est une variante spÃ©cifique oÃ¹ lâattaquant ne reÃ§oit aucun retour dâerreur ou de rÃ©ponse visible directe de la base de donnÃ©es. Lâapplication rÃ©pond alors de maniÃ¨re dÃ©tournÃ©e, soit de faÃ§on **binaire (vrai ou faux)**, soit par un **dÃ©lai dans lâexÃ©cution** de la requÃªte. Cette technique est particuliÃ¨rement redoutable lorsque les messages d'erreur SQL explicites sont dÃ©sactivÃ©s ou filtrÃ©s.

### ð Comparaison des types d'injections
| CritÃ¨re | Injection SQL Classique | Injection SQL Aveugle |
| :--- | :--- | :--- |
| **RÃ©ponse DB** | Messages d'erreurs et rÃ©sultats exploitables | Aucune rÃ©ponse directe de la base |
| **FacilitÃ©** | Facile Ã  dÃ©tecter et Ã  exploiter | Plus difficile, nÃ©cessite des techniques spÃ©cifiques |
| **MÃ©thodes** | Erreurs SQL, affichage direct des donnÃ©es | RÃ©ponses boolÃ©ennes, dÃ©lais d'exÃ©cution |

---

## âï¸ 2. Fonctionnement de l'Attaque
L'attaque repose sur l'observation des changements de comportement de l'application face Ã  des entrÃ©es malveillantes.

### 2.1 Boolean-Based (BasÃ©e sur les boolÃ©ens)
L'attaquant injecte une requÃªte qui force une rÃ©ponse conditionnelle (vrai ou faux).
*   **Test de validitÃ© :** `id=1' AND 1=1 --` (Si la page se charge normalement, la requÃªte est valide).
*   **Test de vulnÃ©rabilitÃ© :** `id=1' AND 1=2 --` (Si la page change de comportement ou affiche une erreur gÃ©nÃ©rique, l'application est vulnÃ©rable).

### 2.2 Time-Based (BasÃ©e sur le temps)
L'attaquant force la base de donnÃ©es Ã  attendre un certain temps avant de rÃ©pondre.
*   **Exemple :** `id=1' AND SLEEP(5) --`.
*   **InterprÃ©tation :** Si la page met exactement 5 secondes de plus Ã  rÃ©pondre, la vulnÃ©rabilitÃ© est confirmÃ©e.

---

## ð ï¸ 3. Exploitation et Exfiltration
L'attaquant identifie la faille en envoyant des caractÃ¨res spÃ©ciaux (`'`, `"`, `--`, `#`) pour repÃ©rer des erreurs masquÃ©es. Une fois confirmÃ©e, il peut extraire des donnÃ©es caractÃ¨re par caractÃ¨re.

### ð Identification de la structure
*   **DÃ©tection d'une table :** `id=1' AND (SELECT COUNT(*) FROM information_schema.tables WHERE table_name='utilisateurs') > 0 --`.
*   **Longueur d'une colonne :** `id=1' AND (SELECT LENGTH(column_name) FROM information_schema.columns WHERE table_name='utilisateurs' LIMIT 1) > 5 --`.

### ð Exfiltration de donnÃ©es
Pour rÃ©cupÃ©rer un contenu (comme un nom d'utilisateur), l'attaquant utilise des tests ASCII :
*   **Exemple :** `id=1' AND ASCII(SUBSTRING((SELECT username FROM utilisateurs LIMIT 1),1,1))=97 --`.
*   Si la page rÃ©pond normalement (Vrai), le premier caractÃ¨re est 'a' (code ASCII 97).

---

## ð 4. Ãtude de Cas : Niveaux de SÃ©curitÃ© (DVWA)
L'analyse de la plateforme **DVWA** montre l'Ã©volution des mesures de protection :

*   **Niveau LOW :** L'entrÃ©e `id` est rÃ©cupÃ©rÃ©e via `$_GET` et injectÃ©e sans aucune validation ni filtrage.
  <img width="940" height="283" alt="image" src="https://github.com/user-attachments/assets/21158ac4-316e-4fbf-9a3e-bea44f5567ce" />

---
*   **Niveau MEDIUM :** Utilisation de `mysql_real_escape_string()`, mais l'absence de guillemets autour de l'ID dans la requÃªte SQL permet de contourner l'Ã©chappement.
  <img width="940" height="267" alt="image" src="https://github.com/user-attachments/assets/97b252be-24d8-462b-9af5-ba00c89257fa" />

---
*   **Niveau HIGH :** L'ID est passÃ© par un cookie (`$_COOKIE`), ce qui oblige l'usage d'outils comme **Burp Suite** pour manipuler les requÃªtes cÃ´tÃ© client.
  <img width="939" height="275" alt="image" src="https://github.com/user-attachments/assets/f14278a1-5ae9-4719-a08c-e346efb92535" />

---
*   **Niveau IMPOSSIBLE :** SÃ©curitÃ© maximale via un typage strict (`intval()`), des **requÃªtes prÃ©parÃ©es (PDO)**, des tokens anti-CSRF et une gestion d'erreur opaque.
  <img width="939" height="510" alt="image" src="https://github.com/user-attachments/assets/00b52b55-9f24-4948-b426-32ba6d0f3ca2" />

---

## ð¡ï¸ 5. MÃ©thodes de Protection et PrÃ©vention
Une protection efficace repose sur plusieurs piliers fondamentaux :

1.  **RequÃªtes PrÃ©parÃ©es (Prepared Statements) :** C'est la dÃ©fense la plus efficace car elle sÃ©pare les donnÃ©es de la structure de la commande SQL.
    *   *Exemple PHP/PDO :* `$stmt = $pdo->prepare("SELECT * FROM utilisateurs WHERE id = :id"); $stmt->execute(['id' => $id]);`.
2.  **Validation et Filtrage :** VÃ©rifier que les entrÃ©es correspondent au format attendu (ex: uniquement des nombres pour un ID) et Ã©chapper les caractÃ¨res dangereux.
3.  **Principe du moindre privilÃ¨ge :** Attribuer uniquement les permissions nÃ©cessaires aux comptes SQL et dÃ©sactiver les fonctions dangereuses comme `LOAD_FILE()` ou `xp_cmdshell`.
4.  **Web Application Firewall (WAF) :** DÃ©tecter et bloquer les patterns d'attaque connus avant qu'ils n'atteignent l'application.

---

## â ï¸ 6. ConsÃ©quences et Risques
Une exploitation rÃ©ussie peut avoir des consÃ©quences dÃ©sastreuses :
*   **AccÃ¨s illimitÃ©** aux donnÃ©es sans authentification.
*   **Vol d'informations sensibles** (identifiants, emails, donnÃ©es personnelles).
*   **ContrÃ´le total du systÃ¨me** via l'exÃ©cution de commandes systÃ¨me.
*   **CoÃ»ts Ã©levÃ©s** de rÃ©paration et dommages irrÃ©parables Ã  la rÃ©putation de l'entreprise.

---

## ð Conclusion
L'injection SQL aveugle est une attaque redoutable car elle peut rester active longtemps sans Ãªtre dÃ©tectÃ©e. Une dÃ©fense robuste nÃ©cessite une approche multicouche combinant code sÃ©curisÃ©, filtrage rigoureux et restrictions des privilÃ¨ges en base de donnÃ©es.


---

<div align="center">

### 🔗 Liens

[![Portfolio](https://img.shields.io/badge/Portfolio-00f2ff?style=for-the-badge&logo=firefox&logoColor=black)](https://kim-san04.github.io) [![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/hakim-sawadogo) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Kim-San04)

**Cheick Abdel Hadime Hakim SAWADOGO**
*Mastère Cybersécurité, Réseaux & Cloud — Efrei Bordeaux*
📧 cheick.sawadogo@efrei.net

![Footer](https://capsule-render.vercel.app/api?type=waving&color=0:00f2ff,100:0d1117&height=80&section=footer)

</div>
