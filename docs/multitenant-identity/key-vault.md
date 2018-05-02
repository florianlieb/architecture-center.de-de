---
title: Verwenden von Key Vault zum Schützen von Anwendungsgeheimnissen
description: Informationen zur Verwendung des Key Vault-Diensts zum Speichern von Anwendungsgeheimnissen
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: d49129a38d0413f6006095f03b817885e1ce6c92
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/16/2018
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="67671-103">Verwenden von Azure Key Vault zum Schützen von Anwendungsgeheimnissen</span><span class="sxs-lookup"><span data-stu-id="67671-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="67671-104">[![GitHub](../_images/github.png)-Beispielcode][sample application]</span><span class="sxs-lookup"><span data-stu-id="67671-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="67671-105">Es kommt häufig vor, dass Anwendungseinstellungen vertraulich sind und geschützt werden müssen. Hierzu gehört z.B. Folgendes:</span><span class="sxs-lookup"><span data-stu-id="67671-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="67671-106">Datenbankverbindungszeichenfolgen</span><span class="sxs-lookup"><span data-stu-id="67671-106">Database connection strings</span></span>
* <span data-ttu-id="67671-107">Kennwörter</span><span class="sxs-lookup"><span data-stu-id="67671-107">Passwords</span></span>
* <span data-ttu-id="67671-108">Kryptografische Schlüssel</span><span class="sxs-lookup"><span data-stu-id="67671-108">Cryptographic keys</span></span>

<span data-ttu-id="67671-109">Aus Sicherheitsgründen sollten Sie diese vertraulichen Daten niemals in der Quellcodeverwaltung speichern.</span><span class="sxs-lookup"><span data-stu-id="67671-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="67671-110">Sie gelangen nur allzu leicht in fremde Hände – auch wenn Ihr Quellcoderepository privat ist.</span><span class="sxs-lookup"><span data-stu-id="67671-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="67671-111">Es geht aber nicht nur darum, geheime Daten vor Fremden zu schützen.</span><span class="sxs-lookup"><span data-stu-id="67671-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="67671-112">Bei größeren Projekten möchten Sie vielleicht einschränken, welche Entwickler und Operatoren Zugriff auf die Produktionsgeheimnisse haben.</span><span class="sxs-lookup"><span data-stu-id="67671-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="67671-113">(Die Einstellungen für Test- und Entwicklungsumgebungen weichen ab.)</span><span class="sxs-lookup"><span data-stu-id="67671-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="67671-114">Eine sicherere Option ist es, diese Geheimnisse in [Azure Key Vault][KeyVault] zu speichern.</span><span class="sxs-lookup"><span data-stu-id="67671-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="67671-115">Key Vault ist ein in der Cloud gehosteter Dienst für die Verwaltung von kryptografischen Schlüsseln und anderer geheimer Schlüssel.</span><span class="sxs-lookup"><span data-stu-id="67671-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="67671-116">Dieser Artikel zeigt, wie Key Vault verwendet wird, um Konfigurationseinstellungen für Ihre App zu speichern.</span><span class="sxs-lookup"><span data-stu-id="67671-116">This article shows how to use Key Vault to store configuration settings for your app.</span></span>

<span data-ttu-id="67671-117">In der [Tailspin Surveys][Surveys]-Anwendung sind folgende Einstellungen vertraulich:</span><span class="sxs-lookup"><span data-stu-id="67671-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="67671-118">die Datenbankverbindungszeichenfolge</span><span class="sxs-lookup"><span data-stu-id="67671-118">The database connection string.</span></span>
* <span data-ttu-id="67671-119">die Redis-Datenbankverbindungszeichenfolge</span><span class="sxs-lookup"><span data-stu-id="67671-119">The Redis connection string.</span></span>
* <span data-ttu-id="67671-120">der geheime Clientschlüssel für die Webanwendung</span><span class="sxs-lookup"><span data-stu-id="67671-120">The client secret for the web application.</span></span>

<span data-ttu-id="67671-121">Die Surveys-Anwendung lädt die Konfigurationseinstellungen aus den folgenden Quellen:</span><span class="sxs-lookup"><span data-stu-id="67671-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="67671-122">aus der Datei „appsettings.json“</span><span class="sxs-lookup"><span data-stu-id="67671-122">The appsettings.json file</span></span>
* <span data-ttu-id="67671-123">Aus dem [Speicher für Benutzergeheimnisse][user-secrets] (nur in Entwicklungsumgebungen, zu Testzwecken)</span><span class="sxs-lookup"><span data-stu-id="67671-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="67671-124">aus der Hostingumgebung (App-Einstellungen in Azure-Web-Apps)</span><span class="sxs-lookup"><span data-stu-id="67671-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="67671-125">Aus Key Vault (wenn aktiviert)</span><span class="sxs-lookup"><span data-stu-id="67671-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="67671-126">Jeder dieser Einstellungen setzt die vorherige außer Kraft, daher haben in Key Vault gespeicherte Einstellungen Vorrang.</span><span class="sxs-lookup"><span data-stu-id="67671-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="67671-127">Standardmäßig ist der Key Vault-Konfigurationsanbieter deaktiviert.</span><span class="sxs-lookup"><span data-stu-id="67671-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="67671-128">Er ist nicht erforderlich, um die Anwendung lokal auszuführen.</span><span class="sxs-lookup"><span data-stu-id="67671-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="67671-129">Sie müssten ihn in einer Produktionsbereitstellung aktivieren.</span><span class="sxs-lookup"><span data-stu-id="67671-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="67671-130">Beim Start liest die Anwendung die Einstellungen aller registrierten Konfigurationsanbieter, und verwendet diese zum Auffüllen eines stark typisierten Optionenobjekts.</span><span class="sxs-lookup"><span data-stu-id="67671-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="67671-131">Weitere Informationen finden Sie unter [Using Options and configuration objects][options] (Verwenden von Optionen und Konfigurationsobjekten).</span><span class="sxs-lookup"><span data-stu-id="67671-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="67671-132">Einrichten von Key Vault in der Surveys-App</span><span class="sxs-lookup"><span data-stu-id="67671-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="67671-133">Voraussetzungen:</span><span class="sxs-lookup"><span data-stu-id="67671-133">Prerequisites:</span></span>

* <span data-ttu-id="67671-134">Installieren Sie die [Azure Resource Manager-Cmdlets][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="67671-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="67671-135">Konfigurieren Sie die Surveys-Anwendung so, wie unter [Ausführen der Surveys-Anwendung][readme] beschrieben.</span><span class="sxs-lookup"><span data-stu-id="67671-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="67671-136">Allgemeine Schritte:</span><span class="sxs-lookup"><span data-stu-id="67671-136">High-level steps:</span></span>

1. <span data-ttu-id="67671-137">Richten Sie einen Benutzer mit Administratorrechten im Mandanten ein.</span><span class="sxs-lookup"><span data-stu-id="67671-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="67671-138">Richten Sie ein Clientzertifikat ein.</span><span class="sxs-lookup"><span data-stu-id="67671-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="67671-139">Erstellen eines Schlüsseltresors</span><span class="sxs-lookup"><span data-stu-id="67671-139">Create a key vault.</span></span>
4. <span data-ttu-id="67671-140">Fügen Sie Ihrem Schlüsseltresor Konfigurationseinstellungen hinzu.</span><span class="sxs-lookup"><span data-stu-id="67671-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="67671-141">Entfernen Sie den Code, der den Schlüsseltresor aktiviert.</span><span class="sxs-lookup"><span data-stu-id="67671-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="67671-142">Aktualisieren Sie die geheimen Benutzerschlüssel der Anwendung.</span><span class="sxs-lookup"><span data-stu-id="67671-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="67671-143">Richten Sie einen Benutzer mit Administratorrechten ein.</span><span class="sxs-lookup"><span data-stu-id="67671-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="67671-144">Um einen Schlüsseltresor zu erstellen, müssen Sie ein Konto verwenden, das Ihr Azure-Abonnement verwalten kann.</span><span class="sxs-lookup"><span data-stu-id="67671-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="67671-145">Jede Anwendung, der Sie erlauben, Daten aus dem Schlüsseltresor zu lesen, muss im gleichen Mandanten wie dieses Konto registriert sein.</span><span class="sxs-lookup"><span data-stu-id="67671-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="67671-146">In diesem Schritt stellen Sie sicher, dass Sie einen Schlüsseltresor erstellen können, während Sie als Benutzer im Mandanten angemeldet sind, in dem auch die Surveys-App registriert ist.</span><span class="sxs-lookup"><span data-stu-id="67671-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="67671-147">Erstellen Sie einen Administratorbenutzer in dem Azure AD-Mandanten, in dem die Surveys-Anwendung registriert ist.</span><span class="sxs-lookup"><span data-stu-id="67671-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="67671-148">Melden Sie sich beim [Azure-Portal][azure-portal] an.</span><span class="sxs-lookup"><span data-stu-id="67671-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="67671-149">Wählen Sie den Azure AD-Mandanten, in dem Ihre Anwendung gespeichert ist.</span><span class="sxs-lookup"><span data-stu-id="67671-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="67671-150">Klicken Sie auf **Weitere Dienste** > **SICHERHEIT + IDENTITÄT** > **Azure Active Directory** > **Benutzer und Gruppen** > **Alle Benutzer**.</span><span class="sxs-lookup"><span data-stu-id="67671-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="67671-151">Klicken Sie oben im Portal auf **Neuer Benutzer**.</span><span class="sxs-lookup"><span data-stu-id="67671-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="67671-152">Füllen Sie die Felder aus, und weisen Sie den Benutzer zur Verzeichnisrolle **Globaler Administrator** hinzu.</span><span class="sxs-lookup"><span data-stu-id="67671-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="67671-153">Klicken Sie auf **Create**.</span><span class="sxs-lookup"><span data-stu-id="67671-153">Click **Create**.</span></span>

![Globaler Administratorbenutzer](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="67671-155">Weisen Sie diesen Benutzer jetzt als Abonnementbesitzer zu.</span><span class="sxs-lookup"><span data-stu-id="67671-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="67671-156">Wählen Sie im Menü „Hub“ die Option **Abonnements** aus.</span><span class="sxs-lookup"><span data-stu-id="67671-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="67671-157">Wählen Sie das Abonnement aus, auf das der Administrator zugreifen soll.</span><span class="sxs-lookup"><span data-stu-id="67671-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="67671-158">Wählen Sie auf dem Blatt „Abonnement“ die Option **Zugriffssteuerung (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="67671-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="67671-159">Klicken Sie auf **Hinzufügen**.</span><span class="sxs-lookup"><span data-stu-id="67671-159">Click **Add**.</span></span>
4. <span data-ttu-id="67671-160">Wählen Sie unter **Rolle** die Option **Besitzer** aus.</span><span class="sxs-lookup"><span data-stu-id="67671-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="67671-161">Geben Sie die E-Mail-Adresse des Benutzers ein, den Sie als Besitzer hinzufügen möchten.</span><span class="sxs-lookup"><span data-stu-id="67671-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="67671-162">Wählen Sie den Benutzer aus, und klicken Sie auf **Speichern**.</span><span class="sxs-lookup"><span data-stu-id="67671-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="67671-163">Einrichten eines Clientzertifikats</span><span class="sxs-lookup"><span data-stu-id="67671-163">Set up a client certificate</span></span>
1. <span data-ttu-id="67671-164">Führen Sie das PowerShell-Skript [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] folgendermaßen aus:</span><span class="sxs-lookup"><span data-stu-id="67671-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="67671-165">Geben Sie für den Parameter `Subject` einen beliebigen Namen ein, z.B. „surveysapp“.</span><span class="sxs-lookup"><span data-stu-id="67671-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="67671-166">Das Skript generiert ein selbstsigniertes Zertifikat und speichert es im Zertifikatspeicher „Aktueller Benutzer/Eigene Zertifikate“.</span><span class="sxs-lookup"><span data-stu-id="67671-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="67671-167">Die Ausgabe des Skripts ist ein JSON-Fragment.</span><span class="sxs-lookup"><span data-stu-id="67671-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="67671-168">Kopieren Sie diesen Wert.</span><span class="sxs-lookup"><span data-stu-id="67671-168">Copy this value.</span></span>

2. <span data-ttu-id="67671-169">Wechseln Sie im [Azure-Portal][azure-portal] zu dem Verzeichnis, in dem die Surveys-Anwendung registriert ist. Wählen Sie dazu in der rechten oberen Ecke des Portals Ihr Konto aus.</span><span class="sxs-lookup"><span data-stu-id="67671-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="67671-170">Klicken Sie auf **Azure Active Directory** > **App-Registrierungen** > Surveys.</span><span class="sxs-lookup"><span data-stu-id="67671-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="67671-171">Klicken Sie auf **Manifest** und dann auf **Bearbeiten**.</span><span class="sxs-lookup"><span data-stu-id="67671-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="67671-172">Fügen Sie die Ausgabe des Skripts in die Eigenschaft `keyCredentials` ein.</span><span class="sxs-lookup"><span data-stu-id="67671-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="67671-173">Es sollte in etwa wie folgt aussehen:</span><span class="sxs-lookup"><span data-stu-id="67671-173">It should look similar to the following:</span></span>
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. <span data-ttu-id="67671-174">Klicken Sie auf **Speichern**.</span><span class="sxs-lookup"><span data-stu-id="67671-174">Click **Save**.</span></span>  

7. <span data-ttu-id="67671-175">Wiederholen Sie die Schritte 3 bis 6, um dasselbe JSON-Fragment zum Anwendungsmanifest der Web-API (Surveys.WebAPI) hinzuzufügen.</span><span class="sxs-lookup"><span data-stu-id="67671-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="67671-176">Führen Sie im PowerShell-Fenster den folgenden Befehl aus, um den Fingerabdruck des Zertifikats abzurufen.</span><span class="sxs-lookup"><span data-stu-id="67671-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="67671-177">Verwenden Sie für `[subject]` den Wert, den Sie im PowerShell-Skript als Antragsteller angegeben haben.</span><span class="sxs-lookup"><span data-stu-id="67671-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="67671-178">Der Fingerabdruck wird unter „Cert Hash(sha1)“ aufgeführt.</span><span class="sxs-lookup"><span data-stu-id="67671-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="67671-179">Kopieren Sie diesen Wert.</span><span class="sxs-lookup"><span data-stu-id="67671-179">Copy this value.</span></span> <span data-ttu-id="67671-180">Sie werden den Fingerabdruck später benötigen.</span><span class="sxs-lookup"><span data-stu-id="67671-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="67671-181">Erstellen eines Schlüsseltresors</span><span class="sxs-lookup"><span data-stu-id="67671-181">Create a key vault</span></span>
1. <span data-ttu-id="67671-182">Führen Sie das PowerShell-Skript [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] folgendermaßen aus:</span><span class="sxs-lookup"><span data-stu-id="67671-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="67671-183">Wenn Sie zur Eingabe von Anmeldeinformationen aufgefordert werden, melden Sie sich mit den Informationen des Azure AD-Benutzers an, die Sie zuvor erstellt haben.</span><span class="sxs-lookup"><span data-stu-id="67671-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="67671-184">Das Skript erstellt eine neue Ressourcengruppe und einen neuen Schlüsseltresor innerhalb dieser Ressourcengruppe.</span><span class="sxs-lookup"><span data-stu-id="67671-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="67671-185">Führen Sie „SetupKeyVault.ps“ erneut wie folgt aus:</span><span class="sxs-lookup"><span data-stu-id="67671-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="67671-186">Legen Sie die folgenden Parameterwerte fest:</span><span class="sxs-lookup"><span data-stu-id="67671-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="67671-187">Schlüsseltresorname = der Name, den Sie dem Schlüsseltresor im vorherigen Schritt zugewiesen haben</span><span class="sxs-lookup"><span data-stu-id="67671-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="67671-188">Surveys-App-ID = die Anwendungs-ID für die Surveys-Webanwendung</span><span class="sxs-lookup"><span data-stu-id="67671-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="67671-189">Surveys.WebApi-ID = die Anwendungs-ID für die Surveys.WebAPI-Anwendung</span><span class="sxs-lookup"><span data-stu-id="67671-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="67671-190">Beispiel:</span><span class="sxs-lookup"><span data-stu-id="67671-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="67671-191">Dieses Skript autorisiert die Web-App und die Web-API, geheime Schlüssel aus Ihrem Schlüsseltresor abzurufen.</span><span class="sxs-lookup"><span data-stu-id="67671-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="67671-192">Weitere Informationen finden Sie unter [Erste Schritte mit Azure Key Vault](/azure/key-vault/key-vault-get-started/).</span><span class="sxs-lookup"><span data-stu-id="67671-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="67671-193">Hinzufügen von Konfigurationseinstellungen zu Ihrem Schlüsseltresor</span><span class="sxs-lookup"><span data-stu-id="67671-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="67671-194">Führen Sie „SetupKeyVault.ps“ wie folgt aus:</span><span class="sxs-lookup"><span data-stu-id="67671-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="67671-195">Hierbei gilt:</span><span class="sxs-lookup"><span data-stu-id="67671-195">where</span></span>
   
   * <span data-ttu-id="67671-196">Schlüsseltresorname = der Name, den Sie dem Schlüsseltresor im vorherigen Schritt zugewiesen haben</span><span class="sxs-lookup"><span data-stu-id="67671-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="67671-197">Redis-DNS-Name = der DNS-Name Ihrer Redis Cache-Instanz</span><span class="sxs-lookup"><span data-stu-id="67671-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="67671-198">Redis-Zugriffsschlüssel = der Zugriffsschlüssel für Ihre Redis Cache-Instanz</span><span class="sxs-lookup"><span data-stu-id="67671-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="67671-199">Jetzt sollten Sie überprüfen, ob die geheimen Schlüssel erfolgreich im Schlüsseltresor gespeichert wurden.</span><span class="sxs-lookup"><span data-stu-id="67671-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="67671-200">Führen Sie den folgenden PowerShell-Befehl aus:</span><span class="sxs-lookup"><span data-stu-id="67671-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="67671-201">Führen Sie „SetupKeyVault.ps“ erneut aus, um die Datenbankverbindungszeichenfolge hinzufügen:</span><span class="sxs-lookup"><span data-stu-id="67671-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="67671-202">wobei `<<DB connection string>>` der Wert der Datenbankverbindungszeichenfolge ist.</span><span class="sxs-lookup"><span data-stu-id="67671-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="67671-203">Für Tests mit der lokalen Datenbank, kopieren Sie die Verbindungszeichenfolge aus der Datei „Tailspin.Surveys.Web/appsettings.json“.</span><span class="sxs-lookup"><span data-stu-id="67671-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="67671-204">Ändern Sie dabei den doppelten umgekehrten Schrägstrich („\\\\“) in einen einfachen umgekehrten Schrägstrich.</span><span class="sxs-lookup"><span data-stu-id="67671-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="67671-205">Der doppelte umgekehrte Schrägstrich ist ein Escapezeichen in der JSON-Datei.</span><span class="sxs-lookup"><span data-stu-id="67671-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="67671-206">Beispiel:</span><span class="sxs-lookup"><span data-stu-id="67671-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="67671-207">Entfernen Sie den Code, der Key Vault aktiviert.</span><span class="sxs-lookup"><span data-stu-id="67671-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="67671-208">Öffnen Sie die Projektmappe „Tailspin.Surveys“.</span><span class="sxs-lookup"><span data-stu-id="67671-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="67671-209">Suchen Sie in „Tailspin.Surveys.Web/Startup.cs“ den folgenden Codeblock, und heben Sie die Auskommentierung auf.</span><span class="sxs-lookup"><span data-stu-id="67671-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="67671-210">Suchen Sie in „Tailspin.Surveys.Web/Startup.cs“ den Code, der den `ICredentialService` registriert.</span><span class="sxs-lookup"><span data-stu-id="67671-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="67671-211">Heben Sie Auskommentierungen für die Zeile mit `CertificateCredentialService` und die Zeile mit `ClientCredentialService` auf:</span><span class="sxs-lookup"><span data-stu-id="67671-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="67671-212">Dieser Änderung ermöglicht es der Web-App, die [Clientassertion][client-assertion] zu verwenden, um OAuth-Zugriffstoken abzurufen.</span><span class="sxs-lookup"><span data-stu-id="67671-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="67671-213">Wenn Sie die Clientassertion verwenden, benötigen Sie keine geheimen OAuth-Clientschlüssel.</span><span class="sxs-lookup"><span data-stu-id="67671-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="67671-214">Alternativ dazu können Sie den geheimen Clientschlüssel im Schlüsseltresor speichern.</span><span class="sxs-lookup"><span data-stu-id="67671-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="67671-215">Sowohl der Schlüsseltresor als auch die Clientassertion verwenden jedoch ein Clientzertifikat. Wenn Sie also den Schlüsseltresor aktivieren, empfiehlt es sich, auch die Clientassertion zu aktivieren.</span><span class="sxs-lookup"><span data-stu-id="67671-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="67671-216">Aktualisierung der geheimen Benutzerschlüssel</span><span class="sxs-lookup"><span data-stu-id="67671-216">Update the user secrets</span></span>
<span data-ttu-id="67671-217">Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf das Tailspin.Surveys.Web-Projekt, und wählen Sie **Benutzerschlüssel verwalten**.</span><span class="sxs-lookup"><span data-stu-id="67671-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="67671-218">Löschen Sie in der Datei „secrets.json“ die vorhandene JSON-Datei, und fügen Sie Folgendes hinzu:</span><span class="sxs-lookup"><span data-stu-id="67671-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

    ```
    {
      "AzureAd": {
        "ClientId": "[Surveys web app client ID]",
        "ClientSecret": "[Surveys web app client secret]",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
        "Asymmetric": {
          "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
          "StoreName": "My",
          "StoreLocation": "CurrentUser",
          "ValidationRequired": "false"
        }
      },
      "KeyVault": {
        "Name": "[key vault name]"
      }
    }
    ```

<span data-ttu-id="67671-219">Ersetzen Sie die Einträge in [eckigen Klammern] durch die korrekten Werte.</span><span class="sxs-lookup"><span data-stu-id="67671-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="67671-220">`AzureAd:ClientId`: die Client-ID der Surveys-App</span><span class="sxs-lookup"><span data-stu-id="67671-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="67671-221">`AzureAd:ClientSecret`: der Schlüssel, der generiert wurde, als Sie die Surveys-Anwendung in Azure AD registriert haben</span><span class="sxs-lookup"><span data-stu-id="67671-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="67671-222">`AzureAd:WebApiResourceId`: der App-ID-URI, den Sie angegeben haben, als Sie die Surveys.WebAPI-Anwendung in Azure AD erstellt haben</span><span class="sxs-lookup"><span data-stu-id="67671-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="67671-223">`Asymmetric:CertificateThumbprint`: der Fingerabdruck des Zertifikats, den Sie erhalten haben, als Sie das Clientzertifikat erstellt haben</span><span class="sxs-lookup"><span data-stu-id="67671-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="67671-224">`KeyVault:Name`: der Name des Ihres Schlüsseltresors</span><span class="sxs-lookup"><span data-stu-id="67671-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="67671-225">`Asymmetric:ValidationRequired` ist „false“, da das Zertifikat, das Sie zuvor erstellt haben, nicht von einer Stammzertifizierungsstelle signiert wurde.</span><span class="sxs-lookup"><span data-stu-id="67671-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="67671-226">Verwenden Sie in Produktionsumgebungen ein Zertifikat, das von einer Stammzertifizierungsstelle signiert wurde, und legen Sie `ValidationRequired` auf „true“ fest.</span><span class="sxs-lookup"><span data-stu-id="67671-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="67671-227">Speichern Sie die aktualisierte secrets.json-Datei.</span><span class="sxs-lookup"><span data-stu-id="67671-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="67671-228">Klicken Sie als nächstes im Projektmappen-Explorer mit der rechten Maustaste auf das Tailspin.Surveys.WebApi-Projekt, und wählen Sie **Geheime Benutzerschlüssel verwalten**.</span><span class="sxs-lookup"><span data-stu-id="67671-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="67671-229">Löschen Sie die vorhandene JSON-Datei, und fügen Sie Folgendes hinzu:</span><span class="sxs-lookup"><span data-stu-id="67671-229">Delete the existing JSON and paste in the following:</span></span>

```
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="67671-230">Ersetzen Sie die Einträge in [eckigen Klammern], und speichern Sie die Datei „secrets.json“.</span><span class="sxs-lookup"><span data-stu-id="67671-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="67671-231">Stellen Sie für die Web-API sicher, dass Sie die Client-ID für die Surveys.WebAPI-Anwendung verwenden, und nicht für die Surveys-Anwendung.</span><span class="sxs-lookup"><span data-stu-id="67671-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="67671-232">[**Weiter**][adfs]</span><span class="sxs-lookup"><span data-stu-id="67671-232">[**Next**][adfs]</span></span>

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
