# phishing_checker
Ce qui a été fait
Détection de phishing à partir de la seule chaîne de l'URL (aucune URL n'est visitée). Base : Mendeley URL dataset (443 148 URLs après nettoyage, 22 % de phishing). Test externe : 48 403 URLs de phishing d'une autre source (Mendeley Phishing URLs), sans domaine commun avec l'entraînement.

Modèle retenu
Modèle empilé : TF-IDF de n-grammes de caractères (3 à 5) + régression logistique, dont la probabilité est donnée à LightGBM avec 18 comptages (longueur, points, mots suspects, IP...). Seuil : 0,6, choisi pour garder un recall élevé tout en réduisant les faux positifs.

Résultats (test, séparation par domaine)
métrique	valeur
accuracy	0,954
precision	0,866
recall	0,941
F1	0,902
ROC-AUC	0,988
rappel externe (phishing d'une autre source)	0,853
Matrice de confusion : VN 62 316, FP 2 771, FN 1 127, VP 17 912. Prédire toujours « légitime » donnerait 77,4 % d'accuracy : l'accuracy seule ne suffit pas.

Démarche
PhiUSIIL écarté (légitimes = domaines nus en HTTPS, biais évident).
Variable is_https retirée (artefact de collecte).
Séparation train/test par domaine pour éviter la fuite.
Retrait du TLD et des extensions de page : F1 0,882 -> 0,862 (moins de dépendance aux signatures de source).
Les 18 comptages seuls (AUC 0,916) sont moins bons que le TF-IDF (0,981), mais l'empilement apporte un gain (F1 0,862 -> 0,891 au seuil 0,5).
Limites
Biais de sélection : les légitimes sont surtout des pages ordinaires. Les mots login, signin, account, verify, secure apparaissent dans 90 à 99,9 % de phishing dans les données. Résultat : accounts.google.com/signin, paypal.com/signin et login.microsoftonline.com étaient classés phishing à 97-99 %. Une liste blanche de 11 domaines corrige ces cas, mais ce n'est pas du ML et elle ne protège pas contre un domaine de confiance compromis.
Le modèle ne connaît pas la réputation d'un domaine : il lit du texte.
Domaines nus courts mal représentés (google.com 39,5 %, github.com 55,6 %).
Le test externe ne contient que du phishing : il mesure le rappel, pas les faux positifs sur une autre source de légitimes. Retirer les domaines communs le rend un peu plus facile.
Le domaine est approximatif (a.site.com != b.site.com) : petite fuite possible. n_subdomains est approximatif pour .co.uk / .com.tn.
Artefact "..." (2 672 URLs, surtout phishing) : cause non vérifiée.
Les tests manuels (URLs écrites à la main) illustrent le comportement, ils ne prouvent rien.
Généralisation fragile entre jeux de données ; les attaquants s'adaptent (URL sans mots suspects, domaines imitant une marque, domaines légitimes compromis).
Les scores mesurent la séparation de deux échantillons, pas la protection d'un utilisateur réel.
Message final
Le modèle est une aide à la décision, pas un verdict : une prédiction n'est pas une preuve.
