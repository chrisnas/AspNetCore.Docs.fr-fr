---
title: Comparer les services gRPC avec les API HTTP
author: jamesnk
description: Découvrez comment gRPC compare avec les API HTTP et ce que sont les scénarios recommandés.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 09/25/2019
uid: grpc/comparison
ms.openlocfilehash: 52b057876481bd9be4f83d93b1f05081ed19660f
ms.sourcegitcommit: a166291c6708f5949c417874108332856b53b6a9
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2019
ms.locfileid: "72589970"
---
# <a name="compare-grpc-services-with-http-apis"></a>Comparer les services gRPC avec les API HTTP

Par [James Newton-King](https://twitter.com/jamesnk)

Cet article explique comment les [services gRPC](https://grpc.io/docs/guides/) sont comparés aux API http (y compris les [API Web](xref:web-api/index)ASP.net Core). La technologie utilisée pour fournir une API pour votre application est un choix important, et gRPC offre des avantages uniques par rapport aux API HTTP. Cet article présente les forces et les faiblesses de gRPC et recommande des scénarios d’utilisation de gRPC sur d’autres technologies.

## <a name="high-level-comparison"></a>Comparaison de haut niveau

Le tableau suivant présente une comparaison de haut niveau des fonctionnalités entre les API gRPC et HTTP avec JSON.

| Fonction          | gRPC                                               | API HTTP avec JSON           |
| ---------------- | -------------------------------------------------- | ----------------------------- |
| Contrat         | Obligatoire ( *. proto*)                                | Facultatif (OpenAPI)            |
| Protocole         | HTTP/2                                             | HTTP                          |
| Charge utile          | [Protobuf (petit, binaire)](#performance)           | JSON (grand, lisible par l’utilisateur)  |
| Prescriptiveness | [Spécification stricte](#strict-specification)      | Compatibilité. Tout HTTP est valide.     |
| Diffusion en continu        | [Client, serveur, bidirectionnel](#streaming)       | Client, serveur                |
| Prise en charge des navigateurs  | [Non (requiert GRPC-Web)](#limited-browser-support) | Oui                           |
| Sécurité         | Transport (TLS)                                    | Transport (TLS)               |
| Génération de code client | [Oui](#code-generation)                      | OpenAPI + outils tiers |

## <a name="grpc-strengths"></a>points forts gRPC

### <a name="performance"></a>Performances

les messages gRPC sont sérialisés à l’aide de [Protobuf](https://developers.google.com/protocol-buffers/docs/overview), un format de message binaire efficace. Protobuf sérialise très rapidement sur le serveur et le client. La sérialisation Protobuf génère des messages de petite taille, importants dans des scénarios de bande passante limitée, tels que les applications mobiles.

gRPC est conçu pour HTTP/2, une révision majeure de HTTP qui offre des avantages significatifs en matière de performances par rapport à HTTP 1. x :

* Tramage et compression binaires. Le protocole HTTP/2 est compact et efficace à la fois pour l’envoi et la réception.
* Multiplexage de plusieurs appels HTTP/2 sur une seule connexion TCP. Le multiplexage élimine [le blocage en tête de ligne](https://en.wikipedia.org/wiki/Head-of-line_blocking).

### <a name="code-generation"></a>Génération de code

Toutes les infrastructures gRPC fournissent une prise en charge de première classe pour la génération de code. Un fichier de base pour le développement gRPC est le [fichier *. proto* ](https://developers.google.com/protocol-buffers/docs/proto3), qui définit le contrat de services et de messages gRPC. À partir de ce fichier, gRPC frameworks code générera une classe de base de service, des messages et un client complet.

En partageant le fichier *. proto* entre le serveur et le client, les messages et le code client peuvent être générés de bout en bout. La génération de code du client élimine la duplication des messages sur le client et le serveur, et crée un client fortement typé pour vous. Le fait de ne pas avoir à écrire un client permet d’économiser beaucoup de temps de développement dans les applications avec de nombreux services.

### <a name="strict-specification"></a>Spécification stricte

Il n’existe aucune spécification formelle pour l’API HTTP avec JSON. Les développeurs ont discuté du meilleur format des URL, des verbes HTTP et des codes de réponse.

La [spécification gRPC](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md) est normative sur le format qu’un service gRPC doit suivre. gRPC élimine le débat et économise le temps des développeurs, car gRPC est cohérent entre les plateformes et les implémentations.

### <a name="streaming"></a>Diffusion en continu

HTTP/2 fournit une base pour les flux de communication de longue durée et en temps réel. gRPC fournit une prise en charge de première classe pour la diffusion via HTTP/2.

Un service gRPC prend en charge toutes les combinaisons de diffusion en continu :

* Unaire (aucune diffusion en continu)
* Streaming de serveur à client
* Diffusion en continu du client vers le serveur
* Streaming bidirectionnel

### <a name="deadlinetimeouts-and-cancellation"></a>Échéance/délais d’attente et annulation

gRPC permet aux clients de spécifier la durée pendant laquelle ils sont prêts à attendre la fin d’un appel de procédure distante (RPC). L' [échéance](https://grpc.io/blog/deadlines) est envoyée au serveur et le serveur peut décider de l’action à entreprendre en cas de dépassement de l’échéance. Par exemple, le serveur peut annuler les demandes de gRPC/HTTP/de base de données en cours d’expiration.

La propagation de l’échéance et de l’annulation via les appels gRPC enfants permet d’appliquer les limites d’utilisation des ressources.

## <a name="grpc-recommended-scenarios"></a>scénarios gRPC recommandés

gRPC est bien adapté aux scénarios suivants :

* Les **microservices** &ndash; gRPC sont conçus pour une communication à faible latence et à débit élevé. gRPC est parfait pour les microservices légers où l’efficacité est essentielle.
* La **communication en temps réel point à point** &ndash; gRPC offre une excellente prise en charge de la diffusion bidirectionnelle. les services gRPC peuvent envoyer des messages en temps réel sans interrogation.
* Les **environnements polyglotte** &ndash; les outils gRPC prennent en charge tous les langages de développement populaires, ce qui fait de gRPC un bon choix pour les environnements multilingues.
* Les **environnements réseau limités** &ndash; les messages gRPC sont sérialisés avec Protobuf, un format de message léger. Un message gRPC est toujours plus petit qu’un message JSON équivalent.

## <a name="grpc-weaknesses"></a>faiblesses gRPC

### <a name="limited-browser-support"></a>Prise en charge limitée du navigateur

Il est impossible d’appeler directement un service gRPC à partir d’un navigateur. gRPC utilise fortement les fonctionnalités HTTP/2 et aucun navigateur ne fournit le niveau de contrôle requis sur les requêtes Web pour prendre en charge un client gRPC. Par exemple, les navigateurs n’autorisent pas un appelant à exiger que le protocole HTTP/2 soit utilisé ou à fournir l’accès aux frames HTTP/2 sous-jacents.

[gRPC-Web](https://grpc.io/docs/tutorials/basic/web.html) est une technologie supplémentaire de l’équipe gRPC qui offre une prise en charge limitée des gRPC dans le navigateur. gRPC-Web se compose de deux parties : un client JavaScript qui prend en charge tous les navigateurs modernes et un proxy gRPC-Web sur le serveur. Le client gRPC-Web appelle le proxy et le proxy transmet les demandes gRPC au serveur gRPC.

Les fonctionnalités de gRPC ne sont pas toutes prises en charge par gRPC-Web. Le client et la diffusion bidirectionnelle ne sont pas pris en charge, et la prise en charge de la diffusion de serveur est limitée.

### <a name="not-human-readable"></a>Non lisible par l’homme

Les demandes de l’API HTTP sont envoyées en tant que texte et peuvent être lues et créées par des humains.

par défaut, les messages gRPC sont encodés avec Protobuf. Bien que Protobuf soit efficace pour l’envoi et la réception, son format binaire n’est pas lisible par l’homme. Protobuf requiert la description d’interface du message spécifiée dans le fichier *. proto* pour une désérialisation correcte. Des outils supplémentaires sont requis pour analyser les charges utiles Protobuf sur le réseau et pour composer les demandes manuellement.

Des fonctionnalités telles que la [réflexion de serveur](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md) et l’outil en [ligne de commande gRPC](https://github.com/grpc/grpc/blob/master/doc/command_line_tool.md) existent pour faciliter les messages binaires Protobuf. En outre, les messages Protobuf prennent en charge la [conversion vers et à partir de JSON](https://developers.google.com/protocol-buffers/docs/proto3#json). La conversion JSON intégrée offre un moyen efficace de convertir des messages Protobuf vers et à partir d’une forme lisible par l’utilisateur lors du débogage.

## <a name="alternative-framework-scenarios"></a>Autres scénarios d’infrastructure

D’autres infrastructures sont recommandées par rapport à gRPC dans les scénarios suivants :

* Les **API accessibles** par le navigateur &ndash; gRPC ne sont pas entièrement prises en charge dans le navigateur. gRPC-Web peut offrir la prise en charge des navigateurs, mais il présente des limitations et introduit un serveur proxy.
* **Communication en temps réel de diffusion** &ndash; gRPC prend en charge la communication en temps réel via la diffusion en continu, mais le concept de diffusion d’un message à des connexions inscrites n’existe pas. Par exemple, dans un scénario de salle de conversation dans lequel de nouveaux messages de conversation doivent être envoyés à tous les clients dans la salle de conversation, chaque appel gRPC est requis pour diffuser individuellement de nouveaux messages de conversation au client. [Signalr](xref:signalr/introduction) est un Framework utile pour ce scénario. Signalr a le concept de connexions persistantes et la prise en charge intégrée de la diffusion des messages.
* La **communication entre processus** &ndash; un processus doit héberger un serveur http/2 pour accepter les appels gRPC entrants. Pour Windows, les [canaux](/dotnet/standard/io/pipe-operations) de communication entre processus sont une méthode de communication rapide et légère.

## <a name="additional-resources"></a>Ressources supplémentaires

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
