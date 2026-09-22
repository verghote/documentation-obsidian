
Le principe est d’utiliser position: sticky sur les cellules du thead, dans un conteneur qui scrolle.
.table-wrapper {
  max-height: 70vh;   /* zone scrollable */
  overflow: auto;
}

table thead th,
table thead td {
  position: sticky;
  top: 0;             /* colle en haut du conteneur scrollable */
  z-index: 3;         /* passe au-dessus du tbody */
  background: #0f7df5;/* fond opaque obligatoire */
}
Points clés :
1.
Le sticky fonctionne par rapport au parent scrollable (overflow: auto|scroll).
2.
Sans top: 0, l’en-tête ne “colle” pas.
3.
Il faut un background (sinon transparence visuelle au scroll).
4.
Mets le sticky sur les vraies cellules d’en-tête (th et td si mélange), pas seulement sur thead.