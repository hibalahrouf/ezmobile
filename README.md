# ezmobile


# Analyse d'APK avec Jadx 
on analysons le mainactivity , la vérification repose sur l’instruction suivante :


il compare  R.string.flag en Base 64 avec l'input de l'utilisateur 
analysons ensuite R.string.flag on a : 
Flag = 0x7f0f0000 donc il pointe sure un 
string stockee dans string.xml avec name Flag



en le decryptopns avec Cyberchef on trouve :





verifions que c'est secret : 

