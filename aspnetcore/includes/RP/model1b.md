<!-- THIS INCLUDE USED BY MVC AND RP -->
Ajoutez les propriétés suivantes à la classe `Movie` :

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/Movie.cs?name=snippet1)]

La classe `Movie` contient :

* Le champ `ID` est nécessaire à la base de données pour la clé primaire.
* `[DataType(DataType.Date)]`:  L’attribut [DataType](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.internal.datatypeattributeadapter) spécifie le type de données (Date). Avec cet attribut :

  * L’utilisateur n’est pas obligé d’entrer les informations de temps dans le champ de date.
  * Seule la date est affichée, pas les informations de temps.

Les [DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) sont traitées dans un prochain didacticiel.