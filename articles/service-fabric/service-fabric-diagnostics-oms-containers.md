---
title: Surveiller les conteneurs dans Azure Service Fabric avec Log Analytics | Microsoft Docs
description: Utilisez Log Analytics pour surveiller les conteneurs qui sont exécutés dans les clusters Azure Service Fabric.
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/1/2017
ms.author: dekapur
ms.openlocfilehash: 7a775b6d23c144c81650bb3608ee6a117475a9ba
ms.sourcegitcommit: 3a4ebcb58192f5bf7969482393090cb356294399
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="monitor-containers-with-log-analytics"></a>Surveiller les conteneurs avec Log Analytics
 
Cet article décrit les étapes requises pour configurer la surveillance de conteneur pour votre cluster. Pour plus d’informations, consultez [Surveillance des conteneurs dans Service Fabric](service-fabric-diagnostics-event-analysis-oms.md#monitoring-containers). Pour obtenir un didacticiel pas à pas de cette procédure, vous pouvez également lire [Surveiller les conteneurs Windows dans Service Fabric](service-fabric-tutorial-monitoring-wincontainers.md).

## <a name="set-up-the-container-monitoring-solution"></a>Configurer la solution de surveillance des conteneurs

> [!NOTE]
> Vous devez configurer Log Analytics pour votre cluster et déployer l’agent OMS sur vos nœuds. Si ce n’est pas le cas, suivez les étapes des rubriques [Configurer Log Analytics](service-fabric-diagnostics-oms-setup.md) et [Ajouter l’agent OMS à un cluster](service-fabric-diagnostics-oms-agent.md).

1. Une fois que votre cluster est configuré avec Log Analytics et l’agent OMS, déployez vos conteneurs. Attendez que vos conteneurs soient déployés avant de passer à l’étape suivante.

2. Dans la Place de marché Azure, recherchez *Solution Container Monitoring*, puis cliquez sur la ressource **Solution Container Monitoring** qui doit s’afficher sous la catégorie Surveillance + Gestion.

    ![Ajout de la solution Conteneurs](./media/service-fabric-diagnostics-event-analysis-oms/containers-solution.png)

3. Créez la solution dans l’espace de travail qui a été créé pour le cluster. Après cette modification, l’agent se met à collecter des données Docker sur les conteneurs. Au bout de 15 minutes environ, la solution doit s’afficher avec les journaux et les statistiques entrants.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur l’orchestration des conteneurs dans Service Fabric : [Service Fabric et conteneurs](service-fabric-containers-overview.md)
* Familiarisez-vous avec les fonctionnalités de [requêtes et recherches dans les journaux](../log-analytics/log-analytics-log-searches.md) offertes dans le cadre de Log Analytics
* Configurez Log Analytics pour paramétrer des règles d’[alerte automatisée](../log-analytics/log-analytics-alerts.md) afin de faciliter la détection et les diagnostics