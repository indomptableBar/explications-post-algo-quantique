<img src="linux-post-quantique.png" align="center"/>


Algorithmes Post-Quantiques pour Certificats sous Linux

Guide pratique pour comprendre, tester et intégrer les algorithmes cryptographiques post-quantiques (PQC)
dans les environnements Linux, notamment pour les certificats, TLS 1.3, SSH et les échanges de clés.




    📋 Sommaire

    À propos

    Pourquoi le post-quantique ?

    Algorithmes couverts

    Standards NIST

    Prérequis

    Installation

    OpenSSL

    liboqs

    OQS-OpenSSL

    ML-DSA et certificats

    ML-KEM et chiffrement hybride

    TLS 1.3

    SSH

    Créer une autorité de certification PQC

    Vérification et diagnostic

    Tailles des clés et signatures

    Bonnes pratiques

    Limites et compatibilité

    Références

    Licence

    📖 À propos

Les Post-Quantum Cryptographic Algorithms (PQC) sont conçus pour rester sûrs face à des ordinateurs quantiques suffisamment puissants.

Les algorithmes cryptographiques actuels tels que RSA et certaines constructions basées sur les courbes elliptiques reposent sur des problèmes mathématiques qui seraient vulnérables à l'algorithme de Shor dans un contexte quantique suffisamment puissant.

Ce projet fournit un guide pratique permettant d'expérimenter avec :

    🔑 ML-KEM — encapsulation de clés post-quantique

    ✍️ ML-DSA — signatures numériques post-quantiques

    🦅 Falcon — signatures basées sur les réseaux

    🌳 SLH-DSA — signatures basées sur le hachage

    🔒 TLS 1.3 hybride

    🖥️ SSH post-quantique

    🏛️ Autorités de certification PQC

    🐧 Intégration dans des environnements Linux

    ⚠️ Ce dépôt est principalement destiné à l'apprentissage, aux tests et à l'évaluation. La disponibilité réelle des algorithmes dépend des versions d'OpenSSL, OpenSSH, des distributions Linux et des logiciels clients/serveurs utilisés.

⚛️ Pourquoi le post-quantique ?
Le problème RSA / ECC

Les systèmes classiques utilisent notamment :

    RSA
     └── Factorisation

    ECC / ECDSA / ECDH
     └── Logarithme discret sur courbes elliptiques

Un ordinateur quantique suffisamment puissant pourrait exploiter l'algorithme de Shor pour résoudre efficacement ces problèmes.

Cela concerne notamment :

    RSA

    Diffie-Hellman classique

    ECDH

    ECDSA

    certaines infrastructures PKI actuelles

Harvest Now, Decrypt Later

Une menace particulièrement importante consiste à :

    Aujourd'hui
         │
         ▼
    Données chiffrées
         │
         │ stockage
         ▼
    ┌──────────────────────┐
    │ Attaquant            │
    │ conserve les données │
    └──────────────────────┘
         │
         │ ordinateur quantique
         ▼
    Déchiffrement futur

Les données nécessitant une confidentialité à long terme peuvent donc justifier une migration cryptographique anticipée.
🔐 Algorithmes couverts
Algorithme	Type	Utilisation

    ML-KEM	KEM	Échange / encapsulation de clés
    ML-DSA	Signature	Certificats, signatures
    SLH-DSA	Signature	Signature basée sur le hachage
    Falcon	Signature	Signatures compactes
    AES-256-GCM	Symétrique	Chiffrement des données
    X25519MLKEM768	KEX hybride	TLS / échange de clés
    ML-KEM

ML-KEM est un mécanisme d'encapsulation de clé (KEM).

Il ne sert pas à chiffrer directement un fichier volumineux.

Le schéma recommandé est plutôt :

                 ML-KEM
                   │
                   ▼
          Secret partagé
                   │
                   ▼
              AES-256-GCM
                   │
                   ▼
                 Données

ML-DSA

ML-DSA est destiné aux signatures numériques.

Il peut être utilisé conceptuellement pour :

certificats

signatures de fichiers

signatures de logiciels

PKI

authentification

SLH-DSA

SLH-DSA est une famille de signatures post-quantiques basée sur le hachage.

Son principal intérêt est de reposer sur des hypothèses cryptographiques différentes de celles utilisées par les constructions basées sur les réseaux.

En contrepartie, les signatures peuvent être beaucoup plus volumineuses.
🏛️ Standards NIST

Les premiers standards PQC du NIST ont été publiés en 2024.

    Standard	Algorithme	Fonction
    FIPS 203	ML-KEM	Encapsulation de clés
    FIPS 204	ML-DSA	Signature numérique
    FIPS 205	SLH-DSA	Signature numérique

    ℹ️ Les noms historiques Kyber, Dilithium et SPHINCS+ sont encore très couramment utilisés dans la documentation et les implémentations. Les noms normalisés NIST sont respectivement ML-KEM, ML-DSA et SLH-DSA.

🐧 Prérequis

Environnement recommandé :

Linux récent

Ubuntu 24.04+

Debian 13+

Fedora 44 ou Fedora 45

accès sudo

GCC / G++

CMake

Git

OpenSSL récent

Vérifier l'environnement :

uname -a
openssl version
ssh -V

📦 Installation
OpenSSL

Les versions récentes d'OpenSSL intègrent progressivement les primitives post-quantiques standardisées.

Vérifier votre version :

openssl version

Sur Debian / Ubuntu :

    sudo apt update
    sudo apt install -y openssl libssl-dev

Compiler une version récente

Si la version fournie par votre distribution n'est pas suffisante :

    sudo apt install -y \
        build-essential \
        perl \
        zlib1g-dev

Puis récupérer une version officielle d'OpenSSL correspondant à votre besoin ici https://openssl-library.org/source/


Exemple :

wget [[https://github.com/openssl/openssl/releases/download/openssl-3.5.0/openssl-3.5.0.tar.gz](https://github.com/openssl/openssl/releases/download/openssl-4.0.2/openssl-4.0.2.tar.gz)]

    tar -xzf openssl-4.0.2.tar.gz
    cd openssl-4.02

    ./Configure \
    --prefix=/usr/local/openssl

    make -j"$(nproc)"

    sudo make install

Vérifier :

    /usr/local/openssl/bin/openssl version

⚠️ Évitez de remplacer brutalement l'OpenSSL système sur une machine de production. De nombreux composants de Linux dépendent de la version fournie par la distribution.

liboqs

Open Quantum Safe fournit notamment liboqs, une bibliothèque destinée à l'expérimentation et à l'évaluation d'algorithmes post-quantiques.

Installer les dépendances :

    sudo apt install -y \
    cmake \
    gcc \
    g++ \
    git \
    libssl-dev

Cloner :

    git clone https://github.com/open-quantum-safe/liboqs.git
    cd liboqs

Compiler :

    mkdir build
    cd build

    cmake \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DBUILD_SHARED_LIBS=ON \
    ..

    make -j"$(nproc)"

    sudo make install
    sudo ldconfig

OQS-OpenSSL

Pour les environnements de test nécessitant des intégrations spécifiques avec OQS :

    git clone https://github.com/open-quantum-safe/openssl.git oqs-openssl

    cd oqs-openssl

    ./Configure \
    --prefix=/usr/local/oqs-openssl

    make -j"$(nproc)"

    sudo make install

⚠️ OQS-OpenSSL et les versions expérimentales ne doivent pas être confondus avec une version standard d'OpenSSL destinée à la production.

✍️ ML-DSA et certificats

Générer une clé ML-DSA

Selon l'implémentation et la version d'OpenSSL utilisée :

    openssl genpkey \
    -algorithm ML-DSA-65 \
    -out server_mldsa65.key

Puis inspecter la clé :

    openssl pkey \
    -in server_mldsa65.key \
    -text \
    -noout

Générer un CSR

    openssl req \
    -new \
    -key server_mldsa65.key \
    -out server_mldsa65.csr \
    -subj "/C=FR/O=MonOrganisme/CN=example.com"

Certificat auto-signé

Pour un environnement de test :

    openssl x509 \
    -req \
    -in server_mldsa65.csr \
    -signkey server_mldsa65.key \
    -out server_mldsa65.crt \
    -days 365

Inspection :

    openssl x509 \
    -in server_mldsa65.crt \
    -text \
    -noout

⚠️ La génération et l'utilisation effective d'un certificat TLS avec une signature PQC nécessitent que toute la chaîne logicielle — serveur, bibliothèque TLS, client et PKI — accepte l'algorithme concerné. La simple présence de l'algorithme dans OpenSSL ne garantit pas son interopérabilité TLS.

🔒 ML-KEM et chiffrement hybride
Principe

ML-KEM est un KEM.

Le principe général est :

Client
  │
  │ clé publique ML-KEM
  ▼
ML-KEM encapsulation
  │
  ├──────────────► ciphertext KEM
  │
  ▼
secret partagé
  │
  ▼
KDF
  │
  ▼
clé AES-256-GCM
  │
  ▼
chiffrement des données

Le destinataire utilise sa clé privée ML-KEM pour récupérer le même secret partagé.
Générer une clé ML-KEM

Avec une implémentation OpenSSL compatible :

    openssl genpkey \
    -algorithm ML-KEM-768 \
    -out kem_priv.pem

Extraire la clé publique :

    openssl pkey \
    -in kem_priv.pem \
    -pubout \
    -out kem_pub.pem

Encapsulation

    openssl pkeyutl \
    -encap \
    -inkey kem_pub.pem \
    -out ciphertext.bin \
    -secret shared_secret.bin

Décapsulation :

    openssl pkeyutl \
    -decap \
    -inkey kem_priv.pem \
    -encaps ciphertext.bin \
    -secret shared_secret_dec.bin

Vérifier :

cmp shared_secret.bin shared_secret_dec.bin && \
    echo "OK : secrets identiques"

⚠️ Chiffrement des données

Un KEM n'est pas destiné à remplacer AES pour chiffrer directement un fichier volumineux.

Une architecture hybride est préférable :

    ML-KEM
       │
       ▼
    Secret partagé
       │
       ▼
      KDF
       │
       ▼
    Clé AES-256-GCM
       │
       ▼
    Fichier / données

Pour une implémentation réelle, utilisez une construction correctement authentifiée avec une KDF adaptée plutôt que de traiter directement le secret KEM comme un mot de passe.
🌐 TLS 1.3

TLS 1.3 peut intégrer des mécanismes post-quantiques au niveau de l'échange de clés lorsque la bibliothèque TLS, le serveur et le client les supportent.

Un exemple de groupe hybride rencontré dans les implémentations modernes est :

X25519MLKEM768

Le concept est :

              TLS 1.3
                 │
          ┌──────┴──────┐
          │             │
       X25519        ML-KEM-768
          │             │
          └──────┬──────┘
                 │
           Secret hybride
                 │
                 ▼
           TLS session keys

Tester avec OpenSSL

Exemple :

    openssl s_client \
        -connect example.com:443 \
        -groups X25519MLKEM768

Pour demander TLS 1.3 :

    openssl s_client \
        -connect example.com:443 \
        -tls1_3 \
        -groups X25519MLKEM768

Inspecter la sortie :

    openssl s_client \
        -connect example.com:443 \
        -tls1_3 \
        -groups X25519MLKEM768 2>&1

⚠️ Le groupe doit être supporté par le client OpenSSL et par le serveur distant. Un nom de groupe présent dans votre installation ne signifie pas nécessairement que le serveur l'acceptera.

🖥️ SSH

Le support PQC dans SSH dépend fortement de la version d'OpenSSH et des patches/extensions utilisés.

Commencer par vérifier :

    ssh -V

Lister les méthodes d'échange disponibles :

    ssh -Q kex

Par exemple :

    ssh -Q kex | grep -i kem

ou :

    ssh -Q kex | grep -Ei "mlkem|kyber|sntrup"

Utiliser un échange hybride

Sur les versions compatibles, OpenSSH peut proposer des méthodes hybrides combinant un mécanisme classique et un mécanisme post-quantique.

Vérifier les algorithmes réellement disponibles :

    ssh -Q kex

Puis tester explicitement une méthode présente dans la liste :

    ssh -vv \
        -o KexAlgorithms=<algorithme-supporté> \
        user@server

    ⚠️ Les commandes ssh-keygen -t ssh-mlkem768 et les options KexAlgorithms présentées dans certaines implémentations expérimentales ne sont pas universelles. Vérifiez toujours ssh -Q kex sur votre version avant de modifier sshd_config.

🏛️ Créer une autorité de certification PQC

Une PKI de test peut être organisée ainsi :

                ┌─────────────────┐
                │    Root CA      │
                │    ML-DSA       │
                └────────┬────────┘
                         │
                         │ signature
                         ▼
                ┌─────────────────┐
                │ Server / TLS    │
                │    ML-DSA       │
                └─────────────────┘

Exemple conceptuel :
1. Clé de la CA

        openssl genpkey \
            -algorithm ML-DSA-87 \
            -out ca_mldsa.key

2. Certificat de la CA

        openssl req \
            -new \
            -x509 \
            -key ca_mldsa.key \
            -out ca_mldsa.crt \
            -days 730 \
            -subj "/C=FR/O=MonOrganisme-CA/CN=Ma CA PQC"

3. Clé du serveur

        openssl genpkey \
            -algorithm ML-DSA-65 \
            -out serveur.key

4. CSR

        openssl req \
        -new \
       -key serveur.key \
       -out serveur.csr \
       -subj "/CN=www.example.com"

5. Signature par la CA

        openssl x509 \
            -req \
            -in serveur.csr \
            -CA ca_mldsa.crt \
            -CAkey ca_mldsa.key \
            -CAcreateserial \
            -out serveur.crt \
            -days 365

6. Vérification

        openssl verify \
        -CAfile ca_mldsa.crt \
        serveur.crt

🔎 Vérification et diagnostic
Version OpenSSL

        openssl version -a

Algorithmes disponibles

    openssl list -public-key-algorithms

KEM :

    openssl list -kem-algorithms

Signature :

    openssl list -signature-algorithms

Rechercher les primitives PQC :

    openssl list -public-key-algorithms | \
        grep -Ei "ML-KEM|ML-DSA|SLH-DSA"

Examiner un certificat

    openssl x509 \
        -in certificate.pem \
        -text \
        -noout

Rechercher l'algorithme de clé publique :

    openssl x509 \
        -in certificate.pem \
        -text \
        -noout | \
        grep -A2 "Public Key Algorithm"

Vérifier une chaîne :

    openssl verify \
        -CAfile ca.pem \
        certificate.pem

📊 Tailles des clés et signatures

Les tailles exactes dépendent notamment du format d'encodage utilisé. Les valeurs ci-dessous sont donc à considérer comme des ordres de grandeur pour comparer les constructions.
Algorithme	Clé publique	Signature
RSA-2048	~256 octets	~256 octets
ECDSA P-256	~65 octets	~64–72 octets
ML-DSA-44	~1,3 Ko	~2,4 Ko
ML-DSA-65	~1,9 Ko	~3,3 Ko
ML-DSA-87	~2,6 Ko	~4,6 Ko
Falcon-512	~0,9 Ko	~0,7 Ko
Falcon-1024	~1,8 Ko	~1,3 Ko
SLH-DSA	variable	nettement plus volumineuse

Les tailles peuvent avoir un impact sur :

   certificats TLS ;

   chaînes de certificats ;

   bande passante ;

   stockage ;

   handshake ;

   performances des serveurs ;

   équipements réseau ;

   systèmes embarqués.

🛡️ Bonnes pratiques
1. Privilégier une migration progressive

Une migration PQC ne consiste pas simplement à remplacer :

RSA → ML-DSA

ou :

ECDH → ML-KEM

Il faut vérifier toute la chaîne :

    Client
       │
       ▼
    Load Balancer
       │
       ▼
    Reverse Proxy
       │
       ▼
    Serveur Web
       │
       ▼
    Bibliothèque TLS
       │
       ▼
    OpenSSL / autre TLS
       │
       ▼
    PKI / CA

2. Tester l'hybride

Pendant la transition, les mécanismes hybrides peuvent combiner des composants classiques et post-quantiques.

Exemple conceptuel :

    Classique
       +   
      PQC
       │
       ▼
    Secret hybride

Cela nécessite toutefois que les deux extrémités supportent exactement la même construction.
3. Ne pas confondre KEM et signature
Fonction	Algorithme
Échange / encapsulation de clé	ML-KEM
Signature	ML-DSA
Signature alternative	SLH-DSA
Chiffrement des données	AES-256-GCM

Ainsi :

ML-KEM ≠ ML-DSA

ML-KEM ne remplace pas directement une signature de certificat.
4. Protéger les clés privées

Les clés privées doivent être protégées :

Permissions strictes
        +
Chiffrement au repos
        +
Gestionnaire de secrets / HSM
        +
Rotation
        +
Sauvegardes sécurisées

Exemple :

    chmod 600 serveur.key

5. Inventorier les certificats

Avant une migration :

┌──────────────────────────────┐
│ Inventaire PKI               │
├──────────────────────────────┤
│ Domaine                      │
│ Algorithme                   │
│ Taille                       │
│ Date expiration              │
│ CA                           │
│ Serveur                      │
│ Clients compatibles          │
└──────────────────────────────┘

L'objectif est d'identifier les dépendances et les systèmes qui ne supportent pas encore les nouvelles primitives.
⚠️ Limites et compatibilité

Le support post-quantique évolue rapidement.

Un algorithme disponible dans :

openssl list

ne signifie pas automatiquement qu'il peut être utilisé dans :

un certificat X.509 ;

une chaîne PKI complète ;

TLS 1.3 ;

Nginx ;

Apache ;

OpenSSH ;

un navigateur ;

un HSM ;

une infrastructure d'entreprise.

Il faut tester l'intégralité de la chaîne d'interopérabilité.

Avant tout déploiement en production :

vérifier la version exacte d'OpenSSL ;

vérifier les algorithmes disponibles ;

vérifier la version du serveur ;

vérifier les clients ;

tester les certificats ;

tester TLS ;

mesurer les performances ;

tester la compatibilité des équipements réseau ;

vérifier les HSM et systèmes de gestion de clés ;

    prévoir une stratégie

