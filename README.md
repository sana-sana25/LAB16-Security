# LAB16-Security : Contournement du SSL Pinning avec Objection et Frida

## Avertissement éthique

Ce laboratoire doit être réalisé uniquement dans un cadre légal et autorisé, sur des applications ou appareils de test.
Le but de ce TP est d’apprendre les techniques d’analyse de sécurité mobile Android et le fonctionnement du SSL Pinning dans les applications mobiles.

---

# Objectifs du laboratoire

Ce laboratoire a pour objectifs :

* Installer et configurer Frida et Objection.
* Configurer un environnement d’analyse réseau Android.
* Mettre en place un proxy HTTPS avec Burp Suite.
* Installer un certificat CA sur Android.
* Désactiver le SSL Pinning d’une application Android.
* Intercepter et analyser le trafic HTTPS de l’application.
* Comprendre le fonctionnement des mécanismes de protection SSL/TLS sur Android.

---

# Environnement de travail

## Machine d’analyse

* Système : Windows
* Outils utilisés :

  * ADB (Android Debug Bridge)
  * Frida
  * Frida-Server
  * Objection
  * Burp Suite

---

## Appareil Android

* Android Emulator
* Android 15
* Débogage USB activé

---

## Application cible

* InsecureShop
<img width="432" height="652" alt="Screenshot 2026-05-22 221005" src="https://github.com/user-attachments/assets/0f2e7724-10b2-4611-add4-02974123f3ae" />

---

# Vérification des prérequis

Avant de commencer, les outils suivants ont été vérifiés :

```bash
python --version
pip --version
adb version
```

Les versions de Frida et Objection ont également été vérifiées :

```bash
objection --version
frida --version
python -c "import frida; print(frida.__version__)"
```

---

# Installation et configuration de Frida

## Identification de l’architecture Android

La commande suivante a été utilisée afin d’identifier l’architecture CPU de l’émulateur :

```bash
adb shell getprop ro.product.cpu.abi
```

---

## Transfert du frida-server

Le serveur Frida a été envoyé vers l’émulateur Android :

```bash
adb push frida-server /data/local/tmp/
```

---

## Attribution des permissions

```bash
adb shell chmod 755 /data/local/tmp/frida-server
```

---

## Démarrage du serveur Frida

Le serveur Frida a été lancé sur l’émulateur :

```bash
adb shell "/data/local/tmp/frida-server &"
```

---

# Résolution du problème de permissions

Lors de l’utilisation d’Objection, une erreur de permissions est apparue :

```text
frida.PermissionDeniedError: unable to access process
```

Le problème a été résolu en exécutant :

```bash
adb root
```

Cette commande a permis de redémarrer ADB avec les privilèges root nécessaires à l’injection Frida.

---

# Vérification de la connexion Frida

Les applications Android visibles par Frida ont été listées avec :

```bash
frida-ps -Uai
```
<img width="1100" height="544" alt="Screenshot 2026-05-22 234903" src="https://github.com/user-attachments/assets/a3a46e21-5877-426e-8f80-786e44d5f79a" />

Cette commande a confirmé que l’émulateur était correctement détecté et que Frida fonctionnait correctement.

---

# Configuration du proxy Burp Suite

## Activation du Proxy Listener

Dans Burp Suite, un Proxy Listener a été configuré sur :

```text
*:8080
```
<img width="1465" height="918" alt="Screenshot 2026-05-22 234641" src="https://github.com/user-attachments/assets/d0a3e84f-d07c-4fb4-96a6-b55bac00522e" />

Le listener a été vérifié en état “Running”.

---

# Configuration du proxy Android

Le proxy réseau de l’émulateur Android a été configuré manuellement avec :

```text
Host : 10.0.2.2
Port : 8080
```

La configuration a également été vérifiée via ADB :

```bash
adb shell settings put global http_proxy 10.0.2.2:8080
adb shell settings get global http_proxy
```
<img width="1867" height="180" alt="Screenshot 2026-05-23 001322" src="https://github.com/user-attachments/assets/1a4e5878-2214-4334-9e83-3def37dffd25" />

Cette configuration permet à l’émulateur Android de rediriger son trafic vers Burp Suite exécuté sur la machine hôte.

---

# Installation du certificat CA Burp

Le certificat Burp a été téléchargé depuis :

```text
http://burp
```
<img width="798" height="740" alt="Screenshot 2026-05-22 211613" src="https://github.com/user-attachments/assets/b8271a2d-6dfa-49af-bbbd-6aec6a5b011d" />

Puis installé sur Android dans les paramètres de sécurité afin d’autoriser l’interception HTTPS.

---

# Résolution du problème réseau

Au début du laboratoire, l’émulateur Android ne disposait pas d’accès Internet.

L’erreur suivante apparaissait dans Chrome Android :

```text
DNS_PROBE_FINISHED_BAD_CONFIG
```

Le problème provenait d’une mauvaise configuration réseau/proxy sur l’émulateur Android.

Après correction :

* l’accès Internet a été rétabli,
* le proxy Burp fonctionnait correctement,
* le trafic HTTPS pouvait être observé.

---

# Validation de l’interception HTTPS

Après la configuration du proxy Burp Suite et l’installation du certificat CA sur l’émulateur Android, plusieurs tests ont été réalisés afin de vérifier le bon fonctionnement de l’interception HTTPS.

Le navigateur Chrome Android a été utilisé pour accéder à des sites HTTPS comme :

```text
https://example.com
https://www.google.com
```

Les requêtes HTTPS ont été correctement interceptées dans Burp Suite.
<img width="1913" height="926" alt="Screenshot 2026-05-23 001646" src="https://github.com/user-attachments/assets/57c63dd6-8373-4666-93b6-63d9edefff8b" />

Les éléments suivants étaient visibles :

* URL des requêtes,
* Méthodes HTTP,
* Headers HTTP/HTTPS,
* Réponses serveur,
* Informations TLS.

Cela a confirmé que :

* le proxy Android était correctement configuré,
* le certificat Burp était accepté par Android,
* l’interception HTTPS fonctionnait correctement,
* le trafic réseau passait bien par Burp Suite.

---

# Lancement de l’application cible

L’application InsecureShop a été lancée sur l’émulateur Android.

Le package Android identifié était :

```text
com.insecureshop
```

---

# Désactivation du SSL Pinning

L’injection Objection a été effectuée avec la commande suivante :

```bash
objection -g com.insecureshop explore --startup-command "android sslpinning disable"
```
<img width="1105" height="459" alt="Screenshot 2026-05-22 235258" src="https://github.com/user-attachments/assets/69d6f90f-3345-4f3d-9426-6813d67494cc" />

---

# Résultat de l’injection

Objection a confirmé l’installation des hooks SSL :

```text
Found com.android.org.conscrypt.TrustManagerImpl
Registering job android-sslpinning-disable
```

Ces messages indiquent que :

* les vérifications SSL Android ont été interceptées,
* le TrustManager Android a été modifié,
* le SSL Pinning a été contourné avec succès.

---

# Observation du trafic réseau

L’application InsecureShop utilisée durant ce laboratoire générait un trafic réseau limité dans l’activité testée.

Cependant, l’environnement d’analyse était entièrement opérationnel :

* Burp Suite interceptait correctement le trafic HTTPS,
* Frida et Objection fonctionnaient correctement,
* le bypass SSL Pinning était actif,
* les connexions HTTPS pouvaient être analysées.

Certaines requêtes HTTPS système Android ont été observées dans Burp Suite :

* [https://example.com](https://example.com)
* [https://www.google.com](https://www.google.com)
* connectivitycheck.gstatic.com
* play.googleapis.com

Le laboratoire a donc permis de valider le fonctionnement complet de la chaîne :

```text
Android Emulator → Burp Suite → Frida → Objection → SSL Pinning Bypass
```

---

# Difficultés rencontrées

## 1. Absence d’Internet sur l’émulateur

Erreur rencontrée :

```text
DNS_PROBE_FINISHED_BAD_CONFIG
```

Cause :

* mauvaise configuration du proxy réseau.

Solution :

* suppression du proxy incorrect,
* reconfiguration correcte du réseau Android Emulator.

---

## 2. Erreur PermissionDenied avec Frida

Erreur :

```text
frida.PermissionDeniedError
```

Cause :

* absence des privilèges root.

Solution :

```bash
adb root
```

---

## 3. Ancienne application incompatible

Certaines APK de laboratoire plus anciennes ont échoué avec :

```text
INSTALL_FAILED_DEPRECATED_SDK_VERSION
```

Cause :

* incompatibilité avec Android 15.

Solution :

* utilisation d’une application Android moderne compatible.

---

# Résultats obtenus

À la fin du laboratoire :

* Frida fonctionnait correctement.
* Objection était capable d’injecter des hooks dans l’application Android.
* Le SSL Pinning a été désactivé avec succès.
* Burp Suite interceptait le trafic HTTPS.
* Le proxy HTTPS Android était opérationnel.

---

# Conclusion

Ce laboratoire a permis d’apprendre les techniques modernes de bypass SSL Pinning sur Android à l’aide de Frida et Objection.

Les différentes étapes ont permis de comprendre :

* le fonctionnement des proxys HTTPS,
* le rôle des certificats CA,
* le fonctionnement interne du SSL Pinning,
* les mécanismes d’instrumentation dynamique Android.

Ce TP constitue une introduction importante aux techniques de sécurité mobile utilisées dans :

* les audits Android,
* le pentest mobile,
* le reverse engineering,
* l’analyse des applications sécurisées.


# Auteur :
ASSEKNOUR SANA - ENSA Marrakech / Cybersecurity
