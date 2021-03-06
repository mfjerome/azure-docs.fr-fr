---
title: Restrictions et problèmes connus relatifs à l’importation d’API dans la gestion des API Azure | Microsoft Docs
description: Détails des problèmes connus et des restrictions relatifs à l’importation dans la gestion des API Azure à l’aide des formats Open API, WSDL ou WADL.
services: api-management
documentationcenter: ''
author: vladvino
manager: vlvinogr
editor: ''
ms.assetid: 7a5a63f0-3e72-49d3-a28c-1bb23ab495e2
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/29/2017
ms.author: apipm
ms.openlocfilehash: ab4bc4024248675c6325159b5507add1274addc9
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/05/2018
---
# <a name="api-import-restrictions-and-known-issues"></a>Restrictions et problèmes connus relatifs à l’importation d’API
## <a name="about-this-list"></a>À propos de cette liste
Quand vous importez une API, vous pouvez rencontrer certaines restrictions ou identifier des problèmes que vous devez résoudre au préalable. Cet article documente ces restrictions et problèmes connus, organisés par format d’importation de l’API.

## <a name="open-api"></a>Open API/Swagger
Si vous recevez des erreurs durant l’importation de votre document Open API, vérifiez que vous l’avez validé à l’aide du concepteur dans le nouveau portail Azure (Conception - Principal - Open API Specification Editor) ou à l’aide d’un outil tiers tel que <a href="http://www.swagger.io">Swagger Editor</a>.

* Seul le format JSON est pris en charge pour OpenAPI.
* Les schémas référencés à l’aide de propriétés **$ref** ne peuvent pas contenir d’autres propriétés **$ref**.
* Les pointeurs **$ref** ne peuvent pas référencer des fichiers externes.
* **x-ms-paths** et **x-servers** sont les seules extensions prises en charge.
* Les extensions personnalisées sont ignorées lors de l’importation et ne sont pas enregistrées ni conservées pour l’exportation.

> [!IMPORTANT]
> Consultez ce [document](https://blogs.msdn.microsoft.com/apimanagement/2018/03/28/important-changes-to-openapi-import-and-export/) pour obtenir des informations importantes et des conseils liés à l’importation OpenAPI.

## <a name="wsdl"></a>WSDL
Les fichiers WSDL servent à générer des API Pass-through SOAP ou constituent le backend d’une API SOAP à REST.

* **WSDL:Import** : le service APM ne prend pas en charge les API utilisant cet attribut. Les clients doivent fusionner les éléments importés dans un seul document.
* **Messages en plusieurs parties** : le service APIM ne prend pas en charge ces types de messages.
* **WCF wsHttpBinding** : les services SOAP créés avec Windows Communication Foundation doivent utiliser basicHttpBinding. wsHttpBinding n’est pas pris en charge.
* **MTOM** : les services utilisant MTOM <em>peuvent</em> fonctionner. Aucune prise en charge officielle n’est disponible pour l’instant.
* Les types de **récursivité** qui sont définis de manière récursive (par exemple, qui font référence à leur propre tableau) ne sont pas pris en charge par APIM.

## <a name="wadl"></a>WADL
Il n’existe aucun problème connu relatif à l’importation WADL.
