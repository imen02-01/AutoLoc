# Stratégie de fetch et de cascade — AutoLoc

| Association | Fetch | Cascade | Justification |
|---|---|---|---|
| Contrat → Paiement | LAZY | ALL + orphanRemoval | Un paiement n'existe que rattaché à son contrat (composition) ; supprimer le contrat supprime ses paiements, et retirer un paiement de la liste le supprime en base. |
| Agence → Vehicule | LAZY | Aucune | Un véhicule survit à la suppression de son agence : il peut être réaffecté. |
| Agence → Employe | LAZY | Aucune | Un employé n'est pas détruit avec son agence ; c'est une entité à part entière. |
| Vehicule ↔ Equipement | LAZY | Aucune | Les équipements sont partagés entre plusieurs véhicules : supprimer un véhicule ne doit pas supprimer un équipement utilisé ailleurs. Un Set évite les doublons et les réinsertions complètes de la table de jointure. |
| Client → Reservation | LAZY | PERSIST | Enregistrer un client enregistre ses nouvelles réservations ; pas de REMOVE, car l'historique des réservations doit être conservé. |
| Reservation → Vehicule | LAZY | Aucune | Un véhicule existe indépendamment de ses réservations. |
| Reservation ↔ Contrat | LAZY | ALL (côté Reservation) | Un contrat n'a de sens que pour sa réservation : il suit son cycle de vie. |
| Vehicule → Maintenance | LAZY | PERSIST | Les maintenances sont enregistrées avec le véhicule ; pas de REMOVE, pour garder l'historique d'entretien. |

# Pourquoi LAZY partout ?

Par défaut, `@ManyToOne` et `@OneToOne` sont EAGER : charger une entité chargerait toute la chaîne d'objets liés. En forçant LAZY, une association n'est chargée que lorsqu'on y accède, ce qui garde les lectures légères. Le prix à payer est une `LazyInitializationException` en cas d'accès hors transaction, traitée dans un atelier ultérieur.