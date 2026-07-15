# Enterprise IPv6 Migration & Dual-Stack Hardening

Projet d'ingénierie réseau modélisant la transition d'une infrastructure d'entreprise IPv4 legacy vers un environnement moderne de transition en double pile (*Dual-Stack*), avec une préparation complète à l'extinction d'IPv4 (*Pure IPv6*).

---

## 🏗️ Synthèse de l'Architecture Réseau

La maquette simule une infrastructure WAN et LAN multi-sites entièrement configurée pour supporter nativement l'adressage IPv6 tout en préservant la compatibilité descendante IPv4 (Dual-Stack) :
* **Plan d'Adressage IPv6** : Déploiement basé sur des adresses globales uniques (GUA) dans le bloc `2001:DB8::/32` (réservé pour la documentation/laboratoire) et d'adresses de liaison locale (LLA) structurées de manière cohérente (ex: `FE80::1` sur les interfaces des passerelles R1).
* **Infrastructure LAN** : Segmentée par VLANs métiers (Direction, RH, Finance, IT) et zones de services de confiance (Zone DNS publique et privée).
* **Infrastructure WAN** : Maillage redondant interconnectant des routeurs de transit (WAN-1, WAN-2, WAN-4) émulant un réseau d'opérateur de télécommunication.

---

## 🛠️ Implémentations Techniques & Sécurisation

### 1. Adressage Dynamique (DHCPv6 Statefull)
* **DHCPv6 Statefull** : Configuration du serveur de distribution DHCPv6 sur le routeur central (`R1`) pour attribuer dynamiquement les GUAs aux hôtes de confiance tout en garantissant la cohérence de l'attribution.
* **Options DNS** : Injection de l'adresse IPv6 du serveur DNS dans les baux distribués aux postes utilisateurs via l'activation des flags d'information (Managed Address Configuration `M` et Other Stateful Configuration `O` via les commandes d'annonce de routeur IPv6 nd).

### 2. Routage Interne & Dynamique (OSPFv3)
* **Routage OSPFv3** : Mise en œuvre du routage dynamique interne IPv6 d'OSPFv3 pour interconnecter de manière résiliente la maille d'opérateur (WAN-1, WAN-2 et WAN-4).
* **Convergence & Redondance** : Les relations de voisinage d'OSPFv3 s'etablissent de manière sécurisée via les adresses de liaison locale (`FE80::/10`), garantissant des chemins de dérivation immédiats vers les serveurs HTTP et de messagerie d'entreprise en cas de rupture de lien.

### 3. Contrôle d'Accès & Hardening (ACLv6)
* **Filtrage IPv6 Strict (ACLs)** : Durcissement des politiques de flux au niveau de la passerelle principale (`R1`) pour restreindre les liaisons inter-VLANs.
* **Hardening du Segment d'Administration (IT)** :
  * Le segment d'administration (VLAN IT) est autorisé à initier des flux de contrôle ICMPv6 (`echo-request`) et de diagnostic vers l'ensemble des autres réseaux de l'entreprise.
  * Une règle de refus explicite et univoque (`deny ipv6 any any`) bloque toutes les tentatives d'analyse et de connexions (dont ICMPv6) initiées depuis les autres réseaux d'utilisateurs non autorisés (VLAN Finance, RH, Direction) à destination de l'IT pour pallier les risques d'élévation latérale de privilèges.

---

## 🧪 Preuves de Recette & Validation des Services

L'intégralité des fonctionnalités requises a fait l'objet d'un audit de validation réussi (preuves capturées dans le répertoire `02_verification_evidences`) :
1. **Voisinage OSPFv3** : Relations de voisinage validées à l'état `FULL` entre les routeurs WAN-1, WAN-2 et WAN-4 (`ospfv3_neighbor_*.png`).
2. **Routage Interne** : Table de routage convergée (`show ipv6 route`) sur R1 confirmant l'apprentissage dynamique de tous les préfixes réseau de l'infrastructure.
3. **Résolution et Accès DNS/HTTP** : Validation d'une requête DNS IPv6 et de la résolution de `web.enterprise.com` vers l'adresse GUA publique, suivie de l'accès sécurisé au site web depuis les navigateurs clients (`dns_resolution_ipv6.png`, `web_access_ipv6.png`).
4. **Audit de la Messagerie Interne** : Envoi et réception fonctionnels de mails en IPv6 (SMTP/POP3) entre les utilisateurs des VLANs clients (`email_received_*.png`).
5. **Tests de Pénétrabilité (ACLv6)** :
  * *Succès* : Le poste d'administration IT parvient à pinguer l'ensemble des actifs réseau (`ping_it_to_finance_success.png`).
  * *Conformité Cybersécurité* : Les tentatives de ping initiées depuis le VLAN Finance vers le VLAN IT sont instantanément rejetées par le routeur R1 via l'ACL IPv6 (`ping_finance_to_it_blocked.png`).

---

## 📂 Organisation des Livrables

```text
.
├── 01_packet_tracer_lab         # Lab Cisco Packet Tracer simulant la transition Dual-Stack / Pure IPv6 (.pkt)
├── 02_verification_evidences    # captures de validation technique d'infrastructure (Audité et Conforme)
│   ├── routing                  # Tables de routage de R1 et états des voisins OSPFv3
│   ├── security                 # Règles d'ACLv6 appliquées et diagnostics de connectivité (pings autorisés/rejetés)
│   └── services                 # États DHCPv6, résolution DNS, navigation Web et échanges de messagerie (SMTP/POP3)
└── 03_technical_documentation   # Spécifications d'infrastructure, historiques CLI et plans d'adressage
