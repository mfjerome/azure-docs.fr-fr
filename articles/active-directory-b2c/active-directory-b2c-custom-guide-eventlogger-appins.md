---
title: Suivre le comportement des utilisateurs à l’aide d’événements dans Application Insights à partir d’Azure AD B2C | Microsoft Docs
description: Guide pas à pas pour activer les journaux d’événements dans Application Insights à partir des parcours utilisateur Azure AD B2C à l’aide de stratégies personnalisées (préversion)
services: active-directory-b2c
documentationcenter: dev-center-name
author: davidmu1
manager: mtillman
ms.service: active-directory-b2c
ms.topic: article
ms.workload: identity
ms.date: 3/15/2018
ms.author: davidmu
ms.openlocfilehash: 28c4cefdd7604dbddf6887dcf494ecea65d658f1
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/05/2018
---
# <a name="track-user-behavior-in-azure-ad-b2c-journeys-by-using-application-insights"></a>Suivre le comportement des utilisateurs dans les parcours Azure AD B2C à l’aide d’Application Insights

Azure Active Directory B2C (Azure AD B2C) fonctionne bien avec Azure Application Insights. Ces services fournissent des journaux d’événements détaillés et personnalisés pour vos parcours utilisateur personnalisés. Cet article explique comment vous mettre en route pour pouvoir : 
* Obtenir des informations détaillées sur le comportement des utilisateurs
* Résoudre les problèmes de vos propres stratégies en développement ou en production
* Mesurer les performances
* Créer des notifications à partir d’Application Insights

> [!NOTE]
> Cette fonctionnalité est en préversion.

## <a name="how-it-works"></a>Fonctionnement
L’infrastructure d’expérience d’identité d’Azure AD B2C inclut désormais le fournisseur `Handler="Web.TPEngine.Providers.UserJourneyContextProvider, Web.TPEngine, Version=1.0.0.0`. Il envoie des données d’événement directement à Application Insights à l’aide de la clé d’instrumentation fournie à Azure AD B2C. 

Un profil technique utilise ce fournisseur pour définir un événement à partir de B2C. Le profil spécifie le nom de l’événement, les revendications qui seront enregistrées et la clé d’instrumentation. Pour publier un événement, le profil technique est ensuite ajouté en tant qu’*étape d’orchestration* ou en tant que *profil technique de validation* dans un parcours utilisateur personnalisé. 

Application Insights peut unifier les événements en utilisant un ID de corrélation pour enregistrer une session utilisateur. Application Insights rend la session et les événements disponibles en quelques secondes et présente de nombreux outils d’analyse, d’exportation et de visualisation.



## <a name="prerequisites"></a>Prérequis

Suivez les étapes décrites dans [Azure Active Directory B2C : bien démarrer avec les stratégies personnalisées](active-directory-b2c-get-started-custom.md). Cet article part du principe que vous utilisez le pack de démarrage de stratégie personnalisée. Cependant, le pack de démarrage n’est pas obligatoire.



## <a name="step-1-create-an-application-insights-resource-and-get-the-instrumentation-key"></a>Étape 1. Créer une ressource Application Insights et récupérer la clé d’instrumentation

Quand vous utilisez Application Insights avec Azure AD B2C, la seule obligation que vous avez est de créer une ressource et d’obtenir une clé d’instrumentation. La création de la ressource se fait dans le [portail Azure](https://portal.azure.com).

1. Dans le portail Azure, au sein de votre locataire d’abonnement, sélectionnez **+ Créer une ressource**. Ce locataire n’est pas votre locataire Azure AD B2C.  
2. Recherchez et sélectionnez **Application Insights**.  
3. Créez une ressource avec **Application web ASP.NET** comme **Type d’Application**dans l’abonnement de votre choix.
4. Une fois la ressource Application Insights créée, ouvrez-la et notez la clé d’instrumentation. 

![Vue d’ensemble d’Application Insights et clé d’instrumentation](./media/active-directory-b2c-custom-guide-eventlogger-appins/app-ins-key.png)

Pour plus d’informations, consultez la [documentation complète sur Application Insights](https://docs.microsoft.com/azure/application-insights/).

## <a name="step-2-add-new-claimtype-definitions-to-your-trust-framework-extension-file"></a>Étape 2. Ajouter de nouvelles définitions ClaimType à votre fichier d’extension d’infrastructure de confiance

Ouvrez le fichier d’extension à partir du pack de démarrage et ajoutez les éléments suivants au nœud `<BuildingBlocks>`. Le nom de ce fichier est généralement `yourtenant.onmicrosoft.com-B2C_1A_TrustFrameworkExtensions.xml`.

```xml

    <ClaimsSchema>
      <ClaimType Id="EventType">
        <DisplayName>EventType</DisplayName>
        <DataType>string</DataType>
        <AdminHelpText />
        <UserHelpText />
      </ClaimType>
      <ClaimType Id="PolicyId">
        <DisplayName>PolicyId</DisplayName>
        <DataType>string</DataType>
        <AdminHelpText />
        <UserHelpText />
      </ClaimType>
      <ClaimType Id="Culture">
        <DisplayName>Culture</DisplayName>
        <DataType>string</DataType>
        <AdminHelpText />
        <UserHelpText />
      </ClaimType>
      <ClaimType Id="CorrelationId">
        <DisplayName>CorrelationId</DisplayName>
        <DataType>string</DataType>
        <AdminHelpText />
        <UserHelpText />
      </ClaimType>
      <!-- Additional claims used for passing claims to the Application Insights provider. -->
      <ClaimType Id="federatedUser">
        <DisplayName>federatedUser</DisplayName>
        <DataType>boolean</DataType>
        <UserHelpText />
      </ClaimType>
      <ClaimType Id="parsedDomain">
        <DisplayName>Parsed Domain</DisplayName>
        <DataType>string</DataType>
        <UserHelpText>The domain portion of the email address.</UserHelpText>
      </ClaimType>
      <ClaimType Id="userInLocalDirectory">
        <DisplayName>userInLocalDirectory</DisplayName>
        <DataType>boolean</DataType>
        <UserHelpText />
      </ClaimType>
    </ClaimsSchema>

```

## <a name="step-3-add-new-technical-profiles-that-use-the-application-insights-provider"></a>Étape 3. Ajouter de nouveaux profils techniques qui utilisent le fournisseur Application Insights

Les profils techniques peuvent être considérés comme des fonctions dans l’infrastructure d’expérience d’identité d’Azure AD B2C. Cet exemple définit cinq profils techniques pour ouvrir une session et publier des événements :

| Profil technique       | Tâche |
| ----------------------------- |:---------------------------------------------|
| AzureInsights-Common | Crée un ensemble commun de paramètres à inclure dans tous les profils techniques `AzureInsights`     | 
| JourneyContextForInsights   | Ouvre la session dans Application Insights et envoie un ID de corrélation |
| AzureInsights-SignInRequest     | Crée un événement `SignIn` avec un ensemble de revendications quand une demande de connexion a été reçue      | 
| AzureInsights-UserSignup | Crée un événement `UserSignup` quand l’utilisateur déclenche l’option d’inscription dans un parcours d’inscription/connexion     | 
| AzureInsights-SignInComplete | Enregistre la réussite d’une authentification quand un jeton a été envoyé à l’application par partie de confiance     | 

Ajoutez les profils au fichier d’extension à partir du pack de démarrage en ajoutant ces éléments au nœud `<ClaimsProviders>`. Le nom de ce fichier est généralement `yourtenant.onmicrosoft.com-B2C_1A_TrustFrameworkExtensions.xml`.

> [!IMPORTANT]
> Modifiez la clé d’instrumentation dans le profil technique `ApplicationInsights-Common` et remplacez-le par le GUID fourni par votre ressource Application Insights.

```xml
<ClaimsProvider>
      <DisplayName>Application Insights</DisplayName>
      <TechnicalProfiles>

        <TechnicalProfile Id="JourneyContextForInsights">
          <DisplayName>Application Insights</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.UserJourneyContextProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="CorrelationId" />
          </OutputClaims>
        </TechnicalProfile>

        <TechnicalProfile Id="AzureInsights-SignInRequest">
          <InputClaims>
            <!-- An input claim with PartnerClaimType="eventName" is required. The Application Insights provider uses it to create an event with the specified value. -->
            <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInRequest" />
          </InputClaims>
          <IncludeTechnicalProfile ReferenceId="AzureInsights-Common" />
        </TechnicalProfile>

        <TechnicalProfile Id="AzureInsights-SignInComplete">
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="SignInComplete" />
            <InputClaim ClaimTypeReferenceId="federatedUser" PartnerClaimType="{property:FederatedUser}" DefaultValue="false" />
            <InputClaim ClaimTypeReferenceId="parsedDomain" PartnerClaimType="{property:FederationPartner}" DefaultValue="Not Applicable" />
          </InputClaims>
          <IncludeTechnicalProfile ReferenceId="AzureInsights-Common" />
        </TechnicalProfile>

        <TechnicalProfile Id="AzureInsights-UserSignup">
          <InputClaims>
            <InputClaim ClaimTypeReferenceId="EventType" PartnerClaimType="eventName" DefaultValue="UserSignup" />
          </InputClaims>
          <IncludeTechnicalProfile ReferenceId="AzureInsights-Common" />
        </TechnicalProfile>

        <TechnicalProfile Id="AzureInsights-Common">
          <DisplayName>Alternate Email</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.Insights.AzureApplicationInsightsProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <!-- The Application Insights instrumentation key that will be used for logging the events. -->
            <Item Key="InstrumentationKey">xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</Item>
            <!-- A Boolean that indicates whether developer mode is enabled. This controls how events are buffered. In a development environment with minimal event volume, enabling developer mode results in events being sent immediately to Application Insights.   -->
            <Item Key="DeveloperMode">false</Item>
            <!-- A Boolean that indicates whether telemetry should be enabled or not.   -->
            <Item Key="DisableTelemetry ">false</Item>
          </Metadata>
          <InputClaims>
            <!-- Properties of an event are added through the syntax {property:NAME}, where NAME is the name of the property being added to the event. DefaultValue can be either a static value or a value that's resolved by one of the supported DefaultClaimResolvers. -->
            <InputClaim ClaimTypeReferenceId="PolicyId" PartnerClaimType="{property:Policy}" DefaultValue="{Policy:PolicyId}" />
            <InputClaim ClaimTypeReferenceId="CorrelationId" PartnerClaimType="{property:JourneyId}" />
            <InputClaim ClaimTypeReferenceId="Culture" PartnerClaimType="{property:Culture}" DefaultValue="{Culture:RFC5646}" />
          </InputClaims>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>

```

## <a name="step-4-add-the-technical-profiles-for-application-insights-as-orchestration-steps-in-an-existing-user-journey"></a>Étape 4. Ajouter les profils techniques pour Application Insights en tant qu’étapes d’orchestration dans un parcours utilisateur existant

Appelez `JournyeContextForInsights` en tant qu’étape d’orchestration 1 :

```xml
<!-- Initialize a session with Application Insights. -->
<OrchestrationStep Order="1" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="JourneyContextForInsights" TechnicalProfileReferenceId="JourneyContextForInsights" />
          </ClaimsExchanges>
        </OrchestrationStep>
```

Appelez `Azure-Insights-SignInRequest` en tant qu’étape d’orchestration 2 pour suivre une demande de connexion/d’inscription reçue :

```xml
<!-- Track that we have received a sign-in request. -->
        <OrchestrationStep Order="2" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="TrackSignInRequest" TechnicalProfileReferenceId="AzureInsights-SignInRequest" />
          </ClaimsExchanges>
        </OrchestrationStep>
```

Immédiatement *avant* l’étape d’orchestration `SendClaims`, ajoutez une nouvelle étape qui appelle `Azure-Insights-UserSignup`. Celle-ci est déclenchée quand l’utilisateur sélectionne le bouton d’inscription dans un parcours d’inscription/connexion.

```xml
        <!-- Handles the user selecting the sign-up link in the local account sign-in page. -->
        <OrchestrationStep Order="9" Type="ClaimsExchange">
          <Preconditions>
            <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
              <Value>newUser</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
              <Value>newUser</Value>
              <Value>false</Value>
              <Action>SkipThisOrchestrationStep</Action>
            </Precondition>
          </Preconditions>
          <ClaimsExchanges>
            <ClaimsExchange Id="TrackUserSignUp" TechnicalProfileReferenceId="AzureInsights-UserSignup" />
          </ClaimsExchanges>
```

Immédiatement après l’étape d’orchestration `SendClaims`, appelez `Azure-Insights-SignInComplete`. Cette étape signale la réussite d’un parcours.

```xml
        <!-- Track that we have successfully sent a token. -->
        <OrchestrationStep Order="11" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="TrackSignInComplete" TechnicalProfileReferenceId="AzureInsights-SignInComplete" />
          </ClaimsExchanges>
        </OrchestrationStep>

```

> [!IMPORTANT]
> Après avoir ajouté les nouvelles étapes d’orchestration, renumérotez les étapes séquentiellement sans sauter d’entiers de 1 à N.


## <a name="step-5-upload-your-modified-extensions-file-run-the-policy-and-view-events-in-application-insights"></a>Étape 5. Charger votre fichier d’extension modifié, exécuter la stratégie et afficher les événements dans Application Insights

Enregistrez et chargez le nouveau fichier d’extension de l’infrastructure de confiance. Ensuite, appelez la stratégie de partie de confiance à partir de votre application ou utilisez l’option **Exécuter maintenant** dans l’interface d’Azure Active Directory B2C. Quelques secondes plus tard, vos événements sont disponibles dans Application Insights.

1. Ouvrez la ressource Application Insights dans votre locataire Azure Active Directory.
2. Sélectionnez **Événements** dans le sous-menu **Utilisation**.
3. Définissez **Pendant** sur **Dernière heure** et **Par** sur **3 minutes**. Vous devrez peut-être sélectionner **Actualiser** pour afficher les résultats.

![Graphique des événements d’utilisation d’Application Insights](./media/active-directory-b2c-custom-guide-eventlogger-appins/app-ins-graphic.png)





## <a name="next-steps"></a>Étapes suivantes

Ajoutez des types de revendication et des événements à votre parcours utilisateur en fonction de vos besoins. Voici une liste des revendications possibles, utilisant des programmes de résolution des revendications supplémentaires.

### <a name="culture-specific-claims"></a>Revendications spécifiques à la culture

```xml
Culture-specific Claims
            Referenced using {Culture:One of the property names below}
 
        /// An enumeration of the claims supported by the <see cref="JourneyCultureDefaultClaimProcessor"/>
        public enum SupportedClaim
        {
             /// The two letter ISO code for the language i.e. en
            LanguageName
             
            /// The two letter ISO code for the region i.e. US
            RegionName,
 
            /// The RFC5646 language code i.e. en-US
            RFC5646,

            /// The LCID of language code i.e. 1033
            LCID
        }

```

### <a name="policy-specific-claims"></a>Revendications spécifiques à la stratégie

```xml
Policy-specific Claims
Referenced using {Policy:One of the property names below}
 
        /// An enumeration of the claims supported by the <see cref="PolicyDefaultClaimProcessor"/> 
        public enum SupportedClaim
        {
            /// The trustframework tenant id
            TrustFrameworkTenantId,
  
            /// The tenant id of the relying party
            RelyingPartyTenantId,
 
            /// The policy id of the policy
            PolicyId,
 
            /// The tenant object id of the policy
            TenantObjectId
}

```

### <a name="openid-connect-specific-claims"></a>Revendications propres à OpenID Connect

```xml
OpenIDConnect Specific Claims
Referenced using {OIDC:One of the property names below}
 
/// 
        /// An enumeration of the claims supported by the <see cref="OpenIdConnectDefaultClaimProcessor"/>

        public enum SupportedClaim
        {
            /// The OpenIdConnect prompt parameter
            Prompt,
 
            /// The OpenIdConnect login_hint parameter
            LoginHint,

            /// The OpenIdConnect domain_hint parameter
            DomainHint,
 
             /// The OpenIdConnect max_age parameter
            MaxAge,
 
            /// The OpenIdConnect client_id parameter
            ClientId,
 
            /// The OpenIdConnect username parameter
            Username,

            /// The OpenIdConnect password parameter
            Password,
 
            /// The OpenIdConnect resource type parameter
            Resource,
 
            /// The OpendIdConext acr_values parameter
             AuthenticationContextReferences
        }
 
```

### <a name="non-protocol-parameters-included-with-oidc-and-oauth2-requests"></a>Paramètres non liés au protocole inclus avec les requêtes OIDC et OAuth2

```xml
Referenced using { OAUTH-KV:Querystring parameter name }
```

Tous les noms de paramètre inclus dans le cadre d’une requête OIDC ou OAuth2 peuvent être mappés à une revendication dans le parcours utilisateur. Vous pouvez ensuite les enregistrer dans l’événement. Par exemple, la requête de l’application peut inclure un paramètre de chaîne de requête avec le nom `app_session`, `loyalty_number` ou `any_string`.

Voici un exemple de requête de l’application :
```
https://login.microsoftonline.com/sampletenant.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_signup_signin&client_id=e1d2612f-c2bc-4599-8e7b-d874eaca1ae1&nonce=defaultNonce&redirect_uri=https%3A%2F%2Fjwt.ms&scope=openid&response_type=id_token&prompt=login&app_session=0a2b45c&loyalty_number=1234567

```
Vous pouvez ensuite ajouter les revendications en ajoutant un élément `InputClaim` à l’événement Application Insights :
```
<InputClaim ClaimTypeReferenceId="app_session" PartnerClaimType="app_session" DefaultValue="{OAUTH-KV:app_session}" />
<InputClaim ClaimTypeReferenceId="loyalty_number" PartnerClaimType="loyalty_number" DefaultValue="{OAUTH-KV:loyalty_number}" />
```

### <a name="other-system-claims"></a>Autres revendications système

Certaines revendications système doivent être ajoutées au jeu de revendications avant d’être disponibles pour être enregistrées en tant qu’événements. Le profil technique `SimpleUJContext` doit être appelé en tant qu’étape d’orchestration ou que profil technique de validation avant que ces revendications ne soient disponibles.

```xml
<ClaimsProvider>
    <DisplayName>User Journey Context Provider</DisplayName>
        <TechnicalProfiles>
            <TechnicalProfile Id="SimpleUJContext">
                <DisplayName>User Journey Context Provide</DisplayName>
                <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.UserJourneyContextProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
                <OutputClaims>
                    <OutputClaim ClaimTypeReferenceId="IP-Address" />
                    <OutputClaim ClaimTypeReferenceId="CorrelationId" />
                    <OutputClaim ClaimTypeReferenceId="DateTimeInUtc" />
                    <OutputClaim ClaimTypeReferenceId="Build" />
                 </OutputClaims>
            </TechnicalProfile>
        </TechnicalProfiles>
</ClaimsProvider>



