# PKI, Certificats X.509 et Signature Électronique 🔐

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenSSL](https://img.shields.io/badge/Tools-OpenSSL-blue.svg)](https://www.openssl.org/)
[![Institution](https://img.shields.io/badge/ENSPY-Génie%20Informatique-green.svg)](https://polytechnique.cm/)

Projet théorique et travaux pratiques réalisés dans le cadre du cours de **Science de l'Information (GIF3025-2)** à l'**École Nationale Supérieure Polytechnique de Yaoundé (ENSPY)**.

---

## 👥 Membres du Groupe (Groupe 30)

* **Nkouathio Sindji Yohan Manel** (24P617)
* **Mbende Alexandra Christine Dorkas** (23P448)
* **Moiffo Tsapgue Maéck-Jospin** (24P219)
* **Kengne Takoukam Chariene Noël** (26Q331)
* **Kadji Akoué Syfried Hurich** (24P146)

**Sous l'encadrement de :** Dr. Hervé TALE KALACHI  
**Année Académique :** 2026 / 2027

---

## 📌 Aperçu du Projet

Ce dépôt contient le rapport d'exposé, la documentation et les scripts de mise en pratique sur la cryptographie à clé publique, le fonctionnement des infrastructures de confiance (PKI) et la validation juridique/technique des documents informatiques.

### 📑 Sommaire du Rapport
1. **Infrastructure à Clé Publique (PKI) :** Composants (CA, RA, OC), cycle de vie d'un certificat et standards PKCS.
2. **Certificat X.509 :** Structures v1/v2/v3, extensions critiques, formats (ASN.1, DER, PEM).
3. **Signature Électronique :** Cadre légal (eIDAS / Loi Camerounaise 2010/021), niveaux de signature et protocole de calcul/vérification d'empreinte.
4. **Atelier Pratique & Cas d'Étude :** Mini-PKI via OpenSSL et analyse du certificat du site officiel de l'ANTIC (`*.antic.cm`).
5. **Limites & Perspectives :** Révocation (CRL/OCSP), automatisation ACME et migration vers la **Cryptographie Post-Quantique (FIPS 204/205 - ML-DSA, SLH-DSA)**.

---

## 🛠️ Atelier Pratique : Mini-PKI OpenSSL

### 1. Génération de la Clé Privée et du Certificat Root CA
```bash
# Génération de la clé privée de l'AC
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out CA.key

# Génération du certificat auto-signé de la CA
openssl req -x509 -new -key CA.key -sha256 -days 3650 \
  -subj "/C=CM/ST=Centre/L=Yaounde/O=Ma CA/CN=exemple.local" \
  -out CA.crt
```

### 2. Demande de Certification (CSR) et Signature du Serveur
```bash
# Génération de la clé serveur et du CSR
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out serveur.key
openssl req -new -key serveur.key -sha256 -out serveur.csr

# Fichier d'extension (ext.cnf)
cat > ext.cnf <<EOF
basicConstraints = critical, CA:FALSE
keyUsage = critical, digitalSignature
extendedKeyUsage = serverAuth
subjectAltName = DNS:www.mon-site.cm, DNS:localhost, IP:127.0.0.1
EOF

# Signature du certificat par la CA
openssl x509 -req -in serveur.csr -CA CA.crt -CAkey CA.key \
  -CAcreateserial -days 90 -sha256 -extfile ext.cnf -out serveur.crt
```

### 3. Signature et Vérification de Fichier
```bash
# Signature du fichier avec la clé privée
echo "Bonjour Bob" > message.txt
openssl dgst -sha256 -sign serveur.key -out message.sig message.txt

# Extraction de la clé publique et vérification
openssl pkey -in serveur.key -pubout -out serveur.pub
openssl dgst -sha256 -verify serveur.pub -signature message.sig message.txt
# Résultat attendu: Verified OK
```
