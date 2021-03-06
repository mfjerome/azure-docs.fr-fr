---
title: Exemples du guide de démarrage rapide de l’interface CLI 2.0 pour Azure Monitor. | Microsoft Docs
description: Exemples de commandes de CLI 2.0 pour accéder aux fonctionnalités d’Azure Monitor. Azure Monitor est un service Microsoft Azure qui vous permet d’envoyer des notifications d’alerte, ou d’appeler des URL web en fonction des valeurs des données de télémétrie configurées ainsi que de mettre à l’échelle automatiquement des services cloud, des machines virtuelles et des applications web.
author: kamathashwin
manager: orenr
editor: ''
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: 1653aa81-0ee6-4622-9c65-d4801ed9006f
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/04/2018
ms.author: ashwink
ms.openlocfilehash: e429ba460a97daed4a7bdf71895fe24c1619a645
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/06/2018
---
# <a name="azure-monitor-cli-20-quick-start-samples"></a>Exemples du guide de démarrage rapide de l’interface CLI 2.0 pour Azure Monitor
Cet article vous montre des exemples de commandes d’interface de ligne de commande (CLI) qui vous permettent d’accéder aux fonctionnalités de surveillance d’Azure Monitor. Azure Monitor permet une mise à l'échelle automatique des services cloud, des machines virtuelles et des applications web, et d’envoyer des notifications d'alerte ou d’appeler des URL web basées sur des valeurs de données de télémétrie configurées.

## <a name="prerequisites"></a>Prérequis


Si vous n’avez pas déjà installé l’interface Azure CLI, suivez les instructions de l’[Installation d’Azure CLI 2.0](/cli/azure/install-azure-cli). Vous pouvez également utiliser [Azure Cloud Shell](/azure/cloud-shell) pour exécuter l’interface CLI en tant qu’expérience interactive dans votre navigateur. 

## <a name="log-in-to-azure"></a>Connexion à Azure
La première étape consiste à vous connecter à votre compte Azure.

```azurecli
az login
```

Après avoir exécuté cette commande, vous devez vous connecter via les instructions à l’écran. Toutes les commandes fonctionnent dans le cadre de votre abonnement par défaut.

Pour répertorier les détails de votre abonnement actuel, utilisez la commande suivante.

```azurecli
az account show
```

Pour remplacer le contexte de travail par un autre abonnement, utilisez la commande suivante.

```azurecli
az account set -s <Subscription ID or name>
```

Pour afficher une liste de toutes les commandes Azure Monitor prises en charge, effectuez les opérations suivantes.

```azurecli
az monitor -h
```

## <a name="view-activity-log-for-a-subscription"></a>Afficher le journal d’activité pour un abonnement

Pour afficher une liste d’événements du journal d’activité, effectuez les opérations suivantes.

```azurecli
az monitor activity-log list
```

Pour voir toutes les options disponibles, essayez les opérations suivantes.

```azurecli
az monitor activity-log list -h
```

Voici un exemple permettant de répertorier les journaux par groupe de ressources

```azurecli
az monitor activity-log list --resource-group <group name>
```

Exemple pour répertorier les journaux par appelant

```azurecli
az monitor activity-log list --caller myname@company.com
```

Exemple pour répertorier les journaux par appelant, sur un type de ressource, dans une plage de dates

```azurecli
az monitor activity-log list --resource-provider Microsoft.Web \
    --caller myname@company.com \
    --start-time 2016-03-08T00:00:00Z \
    --end-time 2016-03-16T00:00:00Z
```

## <a name="work-with-alerts"></a>Utilisation des alertes

Vous pouvez utiliser les informations figurant dans la section pour utiliser les alertes.

### <a name="get-alert-rules-in-a-resource-group"></a>Obtenir des règles d’alerte dans un groupe de ressources

```azurecli
az monitor activity-log alert list --resource-group <group name>
az monitor activity-log alert show --resource-group <group name> --name <alert name>
```

### <a name="create-a-metric-alert-rule"></a>Créer une règle d’alerte métrique

```azurecli
az monitor alert create --name <alert name> --resource-group <group name> \
    --action email <email1 email2 ...> \
    --action webhook <URI> \
    --target <target object ID> \
    --condition "<METRIC> {>,>=,<,<=} <THRESHOLD> {avg,min,max,total,last} ##h##m##s"
```

### <a name="delete-an-alert-rule"></a>Supprimer une règle d’alerte

```azurecli
az monitor alert delete --name <alert name> --resource-group <group name>
```

## <a name="log-profiles"></a>Profils de journal

Utilisez les informations figurant dans cette section pour utiliser les profils de journal.

### <a name="get-a-log-profile"></a>Obtenir un profil de journal

```azurecli
az monitor log-profiles list
az monitor log-profiles show --name <profile name>
```

### <a name="add-a-log-profile-with-retention"></a>Ajouter un profil de journal avec rétention

```azurecli
az monitor log-profiles create --name <profile name> --location <location of profile> \
    --locations <locations to monitor activity in: location1 location2 ...> \
    --categories <categoryName1 categoryName2 ...> \
    --days <# days to retain> \
    --enabled true \
    --storage-account-id <storage account ID to store the logs in>
```

### <a name="add-a-log-profile-with-retention-and-eventhub"></a>Ajouter un profil de journal avec rétention et EventHub

```azurecli
az monitor log-profiles create --name <profile name> --location <location of profile> \
    --locations <locations to monitor activity in: location1 location2 ...> \
    --categories <categoryName1 categoryName2 ...> \
    --days <# days to retain> \
    --enabled true
    --storage-account-id <storage account ID to store the logs in>
    --service-bus-rule-id <service bus rule ID to stream to>
```

### <a name="remove-a-log-profile"></a>Supprimer un profil de journal

```azurecli
az monitor log-profiles delete --name <profile name>
```

## <a name="diagnostics"></a>Diagnostics

Utilisez les informations figurant dans cette section pour utiliser les paramètres de diagnostic.

### <a name="get-a-diagnostic-setting"></a>Obtenir un paramètre de diagnostic

```azurecli
az monitor diagnostic-settings list --resource <target resource ID>
```

### <a name="create-a-diagnostic-log-setting"></a>Créer un paramètre de journal de diagnostic 

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <storage account ID> \
    --resource <target resource object ID> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

### <a name="delete-a-diagnostic-setting"></a>Supprimer un paramètre de diagnostic

```azurecli
az monitor diagnostic-settings delete --name <diagnostic name> \
    --resource <target resource ID>
```

## <a name="autoscale"></a>Autoscale

Utilisez les informations figurant dans cette section pour utiliser les paramètres de mise à l’échelle automatique. Vous devez modifier ces exemples.

### <a name="get-autoscale-settings-for-a-resource-group"></a>Obtenir les paramètres de mise à l’échelle automatique pour un groupe de ressources

```azurecli
az monitor autoscale list --resource-group <group name>
```

### <a name="get-autoscale-settings-by-name-in-a-resource-group"></a>Obtenir les paramètres de mise à l’échelle automatique par nom dans un groupe de ressources

```azurecli
az monitor autoscale show --name <settings name> --resource-group <group name>
```

### <a name="set-auotoscale-settings"></a>Définir les paramètres de mise à l’échelle automatique

```azurecli
az monitor autoscale create --name <settings name> --resource-group <group name> \
    --count <# instances> \
    --resource <target resource ID>
```
