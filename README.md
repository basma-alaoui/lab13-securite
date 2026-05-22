Laboratoire : Contournement de la détection de root sous Android avec Objection
          
           🧰 Outils nécessaires
Outil	Rôle
ADB	Communication avec l’appareil/émulateur
Frida + frida-server	Instrumentation dynamique
Objection	CLI simplifiée pour Frida
App cible	APK avec détection de root (ex. RootBeer, test personnel)
           📦 Prérequis
Appareil Android rooté ou émulateur avec accès root (ex. AVD sans Play Store)
Débogage USB activé
Python 3.8+ et pip
Frida et frida-server de mêmes versions (voir lab précédent)
             🚀 Étapes principales
1. Installer Objection
bash
pip install --user pipx
pipx ensurepath
pipx install objection
Vérification :

bash
objection --version
2. Préparer l’appareil et lancer frida-server
Récupérer l’ABI :

bash
adb shell getprop ro.product.cpu.abi
Télécharger le frida-server correspondant depuis GitHub Frida releases, puis :

bash
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0" &
Vérifier que Frida voit l’appareil :

bash
frida-ps -Uai
3. Démarrer Objection sur l’application cible
bash
objection -g <package_name> explore
Exemple : objection -g com.example.rootcheck explore

4. Désactiver la détection de root
Une fois dans la console Objection :

bash
android root disable
Cette commande hooke automatiquement plusieurs fonctions courantes de détection de root (ex. File.exists, System.getProperty, Runtime.exec, etc.).

5. Valider le bypass
Dans l’application, les alertes de root ne doivent plus apparaître. L’application se comporte comme sur un environnement sain.

6. Automatisation au démarrage
Pour injecter automatiquement le bypass sans taper la commande manuellement :

bash
objection -g <package_name> explore --startup-command "android root disable"
7. Cas particulier : checks natifs
Si l’application utilise des vérifications en C/C++ (bibliothèques natives), la commande android root disable peut être insuffisante. Il faut alors :

Analyser la bibliothèque native (Ghidra)

Écrire un hook Frida personnalisé ciblant la fonction native

Exemple de hook JavaScript :

javascript
Interceptor.attach(Module.findExportByName("libnative.so", "check_root"), {
    onEnter: function(args) { return 0; }
});
             🧪 Exercices pratiques
Téléchargez RootBeer Sample depuis GitHub. Vérifiez la détection de root, puis contournez-la avec Objection.

Créez un petit script Frida personnalisé pour hocker manuellement File.exists("/system/app/Superuser.apk").

Automatisez le lancement d’Objection + bypass au démarrage de l’app via un script shell/batch.
              📊 Résumé des commandes Objection utiles
Commande	Effet
android root disable	Désactive les checks de root les plus courants (Java)
android sslpinning disable	Contourne le certificate pinning
env	Affiche les variables d’environnement
ls / cd	Navigation dans le système de fichiers de l’app
jobs list	Liste les hooks actifs
frida -U -l script.js --no-pause	Injection d’un script personnalisé (de base Frida)
🛠️ Dépannage
Problème	Solution
objection commande introuvable	Ajouter le dossier Scripts de Python au PATH
frida-server se termine tout seul	Lancer avec & ou dans un terminal dédié
android root disable ne fonctionne pas	L’app utilise des checks natives → passer par un hook personnalisé
L’app crash après injection	Vérifier la version de Frida (doit correspondre frida-server)
📚 Références
Objection GitHub

Frida official documentation

OWASP MSTG – Testing Root Detection
