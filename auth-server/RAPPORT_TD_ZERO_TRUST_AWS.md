# Rapport final - TD Zero Trust sur AWS

> Version humanisee, orientee metier + technique  
> Contexte: architecture hybride (on-prem, cloud, SaaS) de Barbichetz Ltd

## 1. Pourquoi ce rapport

L'objectif de ce TD est simple a dire, mais exigeant a bien faire: passer d'une securite basee sur la confiance implicite ("interne = fiable") a une securite Zero Trust ("chaque acces doit etre verifie").

Dans une architecture hybride, les frontieres classiques ont disparu: API exposees, services cloud, collaborateurs mobiles, partenaires externes, outils SaaS. Donc le vrai perimetre de securite devient **l'identite, le contexte, et la verification continue**.

Ce rapport propose une mise en oeuvre realiste sur AWS, avec une logique de priorites et des preuves de controle.

---

## 2. Resume executif

Barbichetz Ltd doit reduire rapidement 4 risques majeurs:

1. Vol d'identifiants (phishing, brute force, credential stuffing)
2. Privileges trop larges ou permanents
3. Exposition excessive des ressources sensibles
4. Manque de tracabilite exploitable (audit, forensic, remediations)

La reponse recommandee est une architecture Zero Trust AWS basee sur:

- verification explicite de chaque acces
- moindre privilege partout (users, services, pipelines)
- segmentation reseau et applicative
- gestion centralisee des logs et detection active des anomalies
- gouvernance continue (revues d'acces, rotation de secrets, patching, KPI)

---

## 3. Contexte et hypothese de travail

### 3.1 Contexte fonctionnel

L'entreprise opere dans un mode hybride:

- applications internes (on-prem)
- services cloud
- applications SaaS
- utilisateurs internes, externes et administrateurs

### 3.2 Contexte securite

Le modele historique (pare-feu + confiance LAN) n'est plus suffisant:

- un compte compromis a l'interieur du SI peut se deplacer lateralement
- les API deviennent la surface d'attaque principale
- les acces privilegies sont une cible prioritaire des attaquants

### 3.3 Exigence du TD

Le sujet demande de couvrir les besoins Zero Trust avec les outils AWS natifs:

- authentification forte
- controle des invites
- privileges temporaires et audites
- protection des apps et donnees sensibles
- controle adapte au risque
- securite des appareils
- detection d'anomalies
- protection reseau
- journalisation complete

---

## 4. Recherche AWS: services pertinents pour Zero Trust

## 4.1 Identite et acces

- **AWS IAM**: politiques fines, roles, principes least privilege
- **IAM Identity Center**: SSO centralise, federation, gestion unifiee des acces
- **MFA**: facteur fort obligatoire sur comptes sensibles
- **STS / roles temporaires**: privileges limites dans le temps
- **AWS Organizations + SCP**: garde-fous globaux multi-comptes

## 4.2 Controle d'acces applicatif et API

- **Amazon API Gateway**: point d'entree, auth, quotas, throttling, validation
- **AWS WAF**: protection web/API (patterns d'attaque, filtrage)
- **AWS Shield**: protection DDoS
- **Amazon CloudFront**: reduction exposition des origines

## 4.3 Reseau et segmentation

- **Amazon VPC**: isolation logique
- **Security Groups / NACL**: controle de flux fin
- **Subnets prives**: ressources sensibles non accessibles directement
- **VPC Endpoints / PrivateLink**: trafic prive vers services AWS
- **AWS Network Firewall**: filtrage avance

## 4.4 Donnees et secrets

- **AWS KMS**: gestion des cles de chiffrement
- **AWS Secrets Manager**: stockage + rotation des secrets
- **RDS en prive**: base de donnees non exposee publiquement
- **Macie (optionnel selon contexte)**: visibilite donnees sensibles

## 4.5 Detection, audit, conformite

- **CloudTrail**: audit des actions AWS
- **CloudWatch**: logs, metriques, alertes
- **GuardDuty**: detection de menaces
- **Security Hub**: vue securite unifiee
- **AWS Config**: conformite continue
- **Audit Manager**: preuves de conformite

---

## 5. Analyse des risques (cas Barbichetz Ltd)

## 5.1 Risques prioritaires

- **R1 - Compte compromis**: acces illegitime a des API sensibles
- **R2 - Privileges excessifs**: impact eleve en cas de compromission
- **R3 - Exposition reseau**: ressources critiques accessibles trop largement
- **R4 - Secrets mal geres**: fuite de credentials et reprise d'acces
- **R5 - Manque de visibilite**: detection tardive et reponse lente

## 5.2 Exemples de scenarios concrets

- Un utilisateur clique sur un phishing, token recupere, API admin abusee
- Un microservice compromis tente de parler a tous les autres services
- Un secret DB expose dans un repo permet extraction de donnees

---

## 6. Reponse Zero Trust: besoin -> controle -> AWS

| Besoin du TD | Controle Zero Trust | Services AWS proposes | Preuve attendue |
|---|---|---|---|
| Authentification forte | MFA + federation + sessions courtes | IAM Identity Center, IAM, MFA | Politique MFA active, logs de connexion |
| Gestion des invites | Acces minimum et expire | IAM roles dedies, revues d'acces | Liste des droits, traces de revue |
| Privileges admin | JIT/JEA + audit | STS roles temporaires, CloudTrail | Historique elevation + expiration |
| Donnees sensibles | Step-up auth + chiffrement | API Gateway, KMS, Secrets Manager | Regles d'acces + cles + rotation |
| Protection identites | Prevention brute force/phishing | WAF, GuardDuty, Security Hub | Alertes + blocages |
| Controle par risque | Conditions contextuelles | IAM conditions, policy based access | Logs de deny/step-up |
| Securite appareils | Device compliance avant acces | Integration IdP/MDM + Verified Access (selon design) | Journal de conformite device |
| Detection anomalies | Monitoring et alerting continu | GuardDuty, CloudWatch, Security Hub | Alertes et incidents traces |
| Protection reseau | Micro-segmentation et prive par defaut | VPC, SG, NACL, PrivateLink | Regles reseau + tests de non acces |
| Journalisation | Tracabilite complete | CloudTrail, CloudWatch Logs | Retention, dashboards, exports audit |

---

## 7. Architecture cible recommandee

### 7.1 Principe global

Tout acces passe par un point de controle. Aucun acces implicite "parce qu'interne".

### 7.2 Flux cible (vue logique)

1. Utilisateur -> authentification SSO + MFA
2. Requete -> API Gateway (authN/authZ, quotas, validation)
3. Filtrage web -> WAF/Shield
4. Services applicatifs en subnets prives
5. Base de donnees privee (pas d'exposition Internet)
6. Secrets via Secrets Manager, chiffrement via KMS
7. Logs et evenements vers CloudWatch + CloudTrail + Security Hub

### 7.3 Regles de confiance

- service-to-service authentifie
- moindre privilege pour chaque role technique
- secrets jamais en clair dans le code
- acces admin temporaire uniquement

---

## 8. Plan de mise en oeuvre (pragmatique)

## Phase 1 - Reduction du risque immediat (2 a 4 semaines)

- MFA obligatoire
- suppression des comptes partages
- secrets externalises (Secrets Manager)
- base de donnees en reseau prive
- logs centralises + alertes critiques

## Phase 2 - Durcissement Zero Trust (4 a 8 semaines)

- API Gateway + WAF en frontal
- politiques d'acces contextuelles
- roles admin JIT avec approbation
- segmentation VPC plus stricte
- GuardDuty + Security Hub operationnels

## Phase 3 - Maturite operationnelle (continu)

- remediations automatiques (EventBridge/Lambda)
- controle conformite continu (AWS Config)
- revues d'acces periodiques et tableaux de bord KPI
- exercices incident response et post-mortem

---

## 9. Indicateurs de succes (KPI)

- % comptes avec MFA active (cible: 100%)
- % ressources sensibles non exposees publiquement
- delai moyen de revocation d'acces
- couverture de journalisation (comptes/services)
- MTTD et MTTR sur incidents securite
- nombre d'acces refuses par politique Zero Trust (signal utile)

---

## 10. Focus "humanise": ce que cela change au quotidien

- Pour les utilisateurs: un peu plus de verification, beaucoup moins de risque de compromission.
- Pour les admins: plus de discipline (roles temporaires), mais une meilleure protection des operations critiques.
- Pour l'entreprise: moins d'angles morts, des preuves d'audit claires, et une meilleure capacite a reagir vite.

Zero Trust n'est pas "tout bloquer". C'est **autoriser intelligemment**, avec des preuves et du contexte.

---

## 11. Annexe technique - Observation sur le `docker-compose.yml` fourni

Le fichier `auth-server/docker-compose.yml` montre plusieurs points a corriger pour etre coherent avec Zero Trust:

- `SPRING_DATASOURCE_USERNAME=root` et mot de passe `root`
- `MYSQL_ROOT_PASSWORD=root`
- secret applicatif en clair (`APP_MASTER_KEY=...`)
- exposition de MySQL via `3307:3306`

### Correctifs recommandes

1. Creer un compte DB applicatif dedie (droits minimaux)
2. Sortir tous les secrets vers un gestionnaire de secrets
3. Eviter l'exposition publique du port DB si non necessaire
4. Activer la rotation des secrets et la tracabilite des acces

Ces corrections sont des gains rapides et tres visibles dans une demarche Zero Trust.

---

## 12. Conclusion

La proposition AWS ci-dessus repond aux attentes du TD: elle transforme une architecture hybride en architecture "verify-first" avec controle continu.

Le benefice principal est double:

- **securite reelle** (moins de confiance implicite, moins de mouvement lateral)
- **securite prouvable** (logs, audit, indicateurs, gouvernance)

En pratique, la reussite depend moins de l'outil que de la constance: politiques claires, revues regulieres, automatisation des controles, et amelioration continue.

