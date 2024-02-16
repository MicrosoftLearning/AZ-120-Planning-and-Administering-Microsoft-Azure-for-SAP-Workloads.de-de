---
lab:
  title: '04b: Implementieren der SAP-Architektur auf Azure-VMs unter Windows'
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 Modul 4: Bereitstellen von SAP in Azure
# Lab 4b: Implementieren der SAP-Architektur auf Azure-VMs unter Windows

Geschätzte Dauer: 150 Minuten

Alle Aufgaben in diesem Lab werden über das Azure-Portal ausgeführt (einschließlich einer PowerShell Cloud Shell-Sitzung).  

   > **Hinweis:** Wenn Sie Cloud Shell nicht verwenden, muss auf dem virtuellen Labcomputer das Az PowerShell-Modul installiert sein [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Labdateien: keine

## Szenario
  
Als Vorbereitung für die Bereitstellung von SAP NetWeaver in Azure möchte die Adatum Corporation eine Demo implementieren, die die Hochverfügbarkeitsimplementierung von SAP NetWeaver auf Azure-VMs veranschaulicht, auf denen die Windows Server 2016 ausgeführt wird.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

-   Bereitstellen von Azure-Ressourcen, die zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung erforderlich sind

-   Konfigurieren des Betriebssystems von Azure-VMs unter Windows zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung

-   Konfigurieren des Clusterings von Azure-VMs unter Windows zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung

## Anforderungen

-   Ein Microsoft Azure-Abonnement mit der ausreichenden Anzahl verfügbarer Dsv3-vCPUs (vier Standard_D2s_v3-VMs mit 2 vCPUs und sechs Standard_D4s_v3-VMs mit jeweils 4 vCPUs) in einer Azure-Region, die Verfügbarkeitszonen unterstützt.

-   Ein Labcomputer mit Azure Cloud Shell-kompatiblem Webbrowser und Zugriff auf Azure


## Übung 1: Bereitstellen von Azure-Ressourcen, die zur Unterstützung von hochverfügbaren SAP NetWeaver-Bereitstellungen erforderlich sind

Duration (Dauer): 60 Minuten

In dieser Übung stellen Sie Azure-Infrastruktur-Computerkomponenten bereit, die zum Konfigurieren von Windows-Clustering erforderlich sind. Dies umfasst das Erstellen eines Paars von Azure-VMs, auf denen Windows Server 2016 ausgeführt wird, in derselben Verfügbarkeitsgruppe.

### Aufgabe 1: Bereitstellen eines Paars von Azure-VMs, auf denen hoch verfügbare Active Directory ausgeführt wird Standard Controller mithilfe einer Azure Resource Manager-Vorlage

1. Starten Sie auf dem Laborcomputer einen Webbrowser, und navigieren Sie zum Azure-Portal unter **https://portal.azure.com**.

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Geschäfts-, Schul- oder Uni- oder persönlichen Microsoft-Konto mit der Besitzer- oder der Mitwirkendenrolle beim Azure-Abonnement an, das Sie für dieses Lab verwenden.

1. Starten Sie im Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal im aktuellen Azure-Abonnement starten, werden Sie aufgefordert, eine Azure-Dateifreigabe zum Speichern von Cloud Shell-Dateien zu erstellen. Akzeptieren Sie in diesem Falls die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Cloud Shell-Bereich die folgenden Befehle aus, um einen flachen Klon des Repositorys zu erstellen, das die Bicep-Vorlage hostet, die Sie für die Bereitstellung eines Azure-VM-Paars verwenden, das hochverfügbare Active Directory-Domänencontroller ausführt, und legen Sie das aktuelle Verzeichnis auf den Speicherort dieser Vorlage und der zugehörigen Parameterdatei fest:

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$rgName` auf `az12001b-ad-RG` festzulegen:

    ```
    $rgName = 'az12003b-ad-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$location` auf den Namen der Azure-Region festzulegen, die Verfügbarkeitszonen unterstützt und in der Sie die Lab-VMs bereitstellen möchten (ersetzen Sie den Platzhalter `<Azure_region>` durch den Namen dieser Region):

    ```
    $location = '<Azure_region>'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um eine Ressourcengruppe namens **az12001b-ad-RG** in der ausgewählten Azure-Region zu erstellen:

    ```
    New-AzResourceGroup -Name $rgName -Location $location
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$deploymentName` festzulegen:

    ```
    $deploymentName = 'az1203b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. Führen Sie im Cloud Shell-Bereich die folgenden Befehle aus, um den Namen des Administratorbenutzerkontos und dessen Kennwort festzulegen (ersetzen Sie die Platzhalter `<username>` und `<password>` durch den Namen des Administratorbenutzerkontos und den Wert des zugehörigen Kennworts):

    ```
    $adminUsername = '<username>'
    $adminPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
    ```

    > **Hinweis:** Stellen Sie sicher, dass das Kennwort die Komplexitätsanforderungen erfüllt, die für die Bereitstellung von Azure-VMs unter Windows gelten (Länge von mindestens 12 Zeichen mit Klein- und Großbuchstaben, Ziffern und Sonderzeichen).

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Bereitstellung auszuführen:

    ```
    New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $rgName -TemplateFile .\main.bicep -TemplateParameterFile .\azuredeploy.parameters.json -adminUsername $adminUsername -adminPassword $adminPassword -c
    ```

1. Überprüfen Sie die Ausgabe des Befehls, und stellen Sie sicher, dass sie keine Fehler und Warnungen enthält. Wenn Sie dazu aufgefordert werden, drücken Sie die **EINGABETASTE**, um mit der Bereitstellung fortzufahren.

    > **Hinweis:** Die Bereitstellung sollte ungefähr 30 Minuten dauern. Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren.

    > **Hinweis:** Wenn die Bereitstellung mit einem Fehler, einschließlich der Anweisung `PowerShell DSC resource MSFT_xADDomainController failed to execute Set-TargetResource functionality with error message: Domain 'adatum.com' could not be found`, fehlschlägt, führen Sie die folgenden Schritte aus, um dieses Problem zu beheben:

    - Navigieren Sie im Azure-Portal zum Blatt der VM **adBDC**. Wählen Sie im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Einstellungen** die Option **Erweiterungen und Anwendungen**, im Bereich **Erweiterungen und Anwendungen** die Option **PrepareBDC** und im Bereich **BDC vorbereiten** die Option **Deinstallieren** aus. 

    - Navigieren Sie zurück zum Blatt der VM **adBDC**, und starten Sie die Azure-VM neu.

    - Navigieren Sie zum Blatt **az1203b-ad-RG**. Wählen Sie im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Einstellungen** die Option **Bereitstellungen** aus.

    - Wählen Sie auf dem Blatt **az1203b-ad-RG \| Bereitstellungen** den Bereitstellungsnamen aus, der mit dem Präfix **az1203b** beginnt, und wählen Sie auf dem Bereitstellungsblatt die Option **Erneut bereitstellen** aus.

    - Geben Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** im Textfeld **Administratorkennwort** dasselbe Kennwort ein, das Sie während der ursprünglichen Bereitstellung verwendet haben, wählen Sie **Überprüfen + erstellen** und dann **Erstellen** aus.

    - Warten Sie nicht, bis die erneute Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die erneute Bereitstellung sollte ungefähr drei Minuten dauern.

### Aufgabe 2: Bereitstellen von Subnetzen, die Azure-VMs hosten, auf denen eine hoch verfügbare SAP NetWeaver-Bereitstellung und der S2D-Cluster ausgeführt werden.

1. Navigieren Sie im Azure-Portal zum Blatt der Ressourcengruppe **az12003b-ad-RG**.

1. Suchen Sie auf dem Blatt der Ressourcengruppe **az12003b-ad-RG** in der Liste der Ressourcen das virtuelle Netzwerk **adVNET**, und klicken Sie auf den Eintrag, um das Blatt **adVNET** anzuzeigen.

1. Navigieren Sie vom Blatt **adVNET** zum zugehörigen Blatt **adVNET – Subnetze**. 

1. Erstellen Sie auf dem Blatt **adVNET – Subnetze** ein neues Subnetz mit den folgenden Einstellungen:

    -   Name: **sapSubnet**

    -   Adressbereiche (CIDR-Block): **10.0.1.0/24**

1. Erstellen Sie auf dem Blatt **adVNET – Subnetze** ein neues Subnetz mit den folgenden Einstellungen:

    -   Name: **s2dSubnet**

    -   Adressbereiche (CIDR-Block): **10.0.2.0/24**

1. Starten Sie im Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal im aktuellen Azure-Abonnement starten, werden Sie aufgefordert, eine Azure-Dateifreigabe zum Speichern von Cloud Shell-Dateien zu erstellen. Akzeptieren Sie in diesem Falls die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die die Ressourcen enthält, die Sie im vorherigen Vorgang bereitgestellt haben:

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um das in der vorherigen Aufgabe erstellte virtuelle Netzwerk zu identifizieren:

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Ressourcen-ID des neu erstellten Subnetzes zu identifizieren:

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1. Kopieren Sie den resultierenden Wert in die Zwischenablage. Sie werden dies in der nächsten Aufgabe benötigen.

### Aufgabe 3: Bereitstellen der Azure Resource Manager-Vorlage zur Bereitstellung von Azure-VMs mit Windows Server 2016, die eine hoch verfügbare SAP NetWeaver-Bereitstellung hosten

1. Suchen Sie auf dem Labcomputer im Azure-Portal nach der Option **Vorlagenbereitstellung (Bereitstellen mit benutzerdefinierten Vorlagen)**, und wählen Sie sie aus.

1. Geben Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** in der Dropdownliste **Schnellstartvorlage (Haftungsausschluss)** den Text **application-workloads/sap/sap-3-tier-marketplace-image-md** ein, und klicken Sie auf **Vorlage auswählen**.

    > **Hinweis:** Stellen Sie sicher, dass Sie Microsoft Edge oder einen Browser eines Drittanbieters verwenden. Verwenden Sie nicht Internet Explorer.

1. Wählen Sie auf dem Blatt **SAP NetWeaver der Ebene 3 (verwalteter Datenträger)** die Option **Vorlage bearbeiten** aus.

1. Wenden Sie auf dem Blatt **Vorlage bearbeiten** die folgende Änderung an, und wählen Sie **Speichern** aus:

    -   Ersetzen Sie in Zeile **197** `"dbVMSize": "Standard_E8s_v3",` durch `"dbVMSize": "Standard_D4s_v3",`.

1. Geben Sie auf dem Blatt **SAP NetWeaver der Ebene 3 (verwalteter Datenträger)** die folgenden Einstellungen an, klicken Sie auf **Überprüfen + erstellen**, und klicken Sie dann auf **Erstellen**, um die Bereitstellung zu initiieren:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Name einer neuen Ressourcengruppe* **az12003b-sap-RG** |
    | **Location** | *dieselbe Azure-Region, die Sie in der ersten Aufgabe dieser Übung angegeben haben* |
    | **SAP-System-ID** | **I20** |
    | **Stapeltyp** | **ABAP** |
    | **Betriebssystemtyp** | **Windows Server 2016 Datacenter** |
    | **Dbtype** | **SQL** |
    | **SAP-Systemgröße** | **Demo** |
    | **Systemverfügbarkeit** | **Hochverfügbarkeit** |
    | **Benutzername des Administrators** | **Kursteilnehmer** |
    | **Authentifizierungstyp** | **Kennwort** |
    | **Administratorkennwort oder -schlüssel** | *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben* |
    | **Subnetz-ID** | *der Wert, den Sie in der vorherigen Aufgabe in die Zwischenablage kopiert haben* |
    | **Verfügbarkeitszonen** | **1,2** |
    | **Location** | **[resourceGroup().location]** |
    | **„_artifacts“-Speicherort** | **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/** |
    | **SAS-Token für „_artifacts“-Speicherort** | *Leer lassen* |

1. Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. 

### Aufgabe 4: Bereitstellen des Dateiserverclusters mit horizontaler Hochskalierung (Scale-Out File Server, SOFS)

In dieser Aufgabe stellen Sie den Dateiservercluster mit horizontaler Hochskalierung (Scale-Out File Server, SOFS) bereit, der eine Dateifreigabe für die SAP ASCS-Server hosten wird, indem Sie eine Azure Resource Manager-Schnellstartvorlage von GitHub verwenden, die bei GitHub unter [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md) verfügbar ist. 

1. Starten Sie auf dem Laborcomputer einen Browser, und navigieren Sie zu [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md). 

    > **Hinweis:** Stellen Sie sicher, dass Sie Microsoft Edge oder einen Browser eines Drittanbieters verwenden. Verwenden Sie nicht Internet Explorer.

1. Klicken Sie auf der Seite **Erstellen einer Instanz von Direkte Speicherplätze (S2D) für SOFS-Cluster (Dateiserver mit horizontaler Skalierung) mit Windows Server 2016 mithilfe von Managed Disks** auf **In Azure bereitstellen**. Daraufhin wird Ihr Browser automatisch zum Azure-Portal umgeleitet und zeigt das Blatt **Benutzerdefinierte Bereitstellung** an.

1. Geben Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** die folgenden Einstellungen an, klicken Sie auf **Überprüfen + erstellen**, und klicken Sie dann auf **Erstellen**, um die Bereitstellung zu initiieren:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Name einer neuen Ressourcengruppe* **az12003b-s2d-RG** |
    | **Region** | *dieselbe Azure-Region, in der Sie Azure-VMs in den vorherigen Aufgaben dieser Übung bereitgestellt haben* |
    | **Namenspräfix** | **i20** |
    | **VM-Größe** | **Standard D4s\_v3** |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **true** |
    | **Image-SKU** | **2016-Datacenter-Server-Core** |
    | **VM-Anzahl** | **2** |
    | **VM-Datenträgergröße** | **128** |
    | **VM-Datenträgeranzahl** | **3** |
    | **Vorhandener Domänenname** | **adatum.com** |
    | **Benutzername des Administrators** | **Kursteilnehmer** |
    | **Administratorkennwort** | *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben* |
    | **RG-Name des vorhandenen virtuellen Netzwerks** | **az12003b-ad-RG** |
    | **Name des vorhandenen virtuellen Netzwerks** | **adVNet** |
    | **Name des vorhandenen Subnetzes** | **s2dSubnet** |
    | **SOFS-Name** | **sapglobalhost** |
    | **Freigabename** | **sapmnt** |
    | **Tag der geplanten Aktualisierung** | **Sonntag** |
    | **Uhrzeit der geplanten Aktualisierung** | **3:00 Uhr** |
    | **Echtzeit-Antischadsoftware aktiviert** | **false** |
    | **Geplante Antischadsoftware aktiviert** | **false** |
    | **Uhrzeit der geplanten Antischadsoftware** | **120** |
    | **„_artifacts“-Speicherort** | **https://raw.githubusercontent.com/polichtm/301-storage-spaces-direct-md/master** |
    | **SAS-Token für „_artifacts“-Speicherort** | **Behalten Sie den Standardwert bei.** |

1. Die Bereitstellung dauert etwa 20 Minuten. Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort.

    > **Hinweis:** Wenn die Bereitstellung während der Bereitstellung der Komponente „i20-s2d-1/s2dPrep“ oder „i20-s2d-0/s2dPrep“ mit der Fehlermeldung **Konflikt** fehlschlägt, führen Sie die folgenden Schritte aus, um dieses Problem zu beheben:

       - Navigieren Sie im Azure-Portal zur VM **i20-s2d-0**. Wählen Sie im vertikalen Navigationsmenü im Abschnitt **Vorgänge** die Option **Befehl ausführen** aus. Geben Sie im Bereich **Befehlsskript ausführen** in das Textfeld **PowerShell-Skript** das folgende Skript ein, und wählen Sie dann die Schaltfläche **Ausführen** aus (achten Sie darauf, den Platzhalter `<password>` durch das Kennwort zu ersetzen, das Sie zuvor in diesem Lab angegeben haben):

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```

       - Navigieren Sie zum Blatt der VM **i20-s2d-1**. Wählen Sie im vertikalen Navigationsmenü im Abschnitt **Vorgänge** die Option **Befehl ausführen** aus. Geben Sie im Bereich **Befehlsskript ausführen** in das Textfeld **PowerShell-Skript** das folgende Skript ein, und wählen Sie dann die Schaltfläche **Ausführen** aus (achten Sie darauf, den Platzhalter `<password>` durch das Kennwort zu ersetzen, das Sie zuvor in diesem Lab angegeben haben):

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```
       
       - Wiederholen Sie die Schritte der aktuellen Aufgabe von Anfang an.

### Aufgabe 5: Bereitstellen eines Bastion Host

   > **Hinweis:** Da Azure-VMs, die Sie in der vorherigen Aufgabe bereitgestellt haben, über das Internet nicht zugänglich sind, stellen Sie einen virtuellen Azure-Computer mit Windows Server 2016 Datacenter bereit, der als Bastion Host fungiert. 

1. Klicken Sie auf dem Labcomputer in der Benutzeroberfläche des Azure-Portals auf **+ eine Ressource erstellen**.

1. Initiieren Sie auf dem Blatt **Neu** die Erstellung einer neuen Azure-VM basierend auf dem **Windows Server 2019 Datacenter – Gen1**-Image.

1. Stellen Sie eine Azure-VM mit den folgenden Einstellungen bereit:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Name einer neuen Ressourcengruppe*, **az12003b-dmz-RG** |
    | **Name des virtuellen Computers** | **az12003b-vm0** |
    | **Region** | *dieselbe Azure-Region, in der Sie Azure-VMs in den vorherigen Aufgaben dieser Übung bereitgestellt haben* |
    | **Verfügbarkeitsoptionen** | **Keine Infrastrukturredundanz erforderlich** |
    | **Image** | *wählen Sie* **Windows Server 2019 Datacenter – Gen2** aus |
    | **Größe** | **Standard D2s_v3** |
    | **Benutzername** | **Kursteilnehmer** |
    | **Kennwort** | *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben* |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Möchten Sie eine vorhandene Windows Server-Lizenz verwenden?** | **Nein** |
    | **Typ des Betriebssystemdatenträgers** | **HDD Standard** |
    | **Virtuelles Netzwerk** | **adVNET** |
    | **Subnetzname** | *ein neues Subnetz namens* **dmzSubnet** |
    | **Subnetzadressbereich** | **10.0.255.0/24** |
    | **Öffentliche IP-Adresse** | *eine neue IP-Adresse namens* **az12003b-vm0-ip** |
    | **NIC-Netzwerksicherheitsgruppe** | **Grundlegend**  |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Deaktiviert** |
    | **Optionen für den Lastenausgleich** | **Keine** |
    | **Aktivieren einer systemseitig zugewiesenen verwalteten Identität** | **Deaktiviert** |
    | **Mit Azure AD anmelden** | **Deaktiviert** |
    | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
    | **Optionen zur Patchorchestrierung** | **Manuelle Updates** |
    | **Startdiagnose** | **Deaktivieren** |
    | **Diagnose des Gastbetriebssystems aktivieren** | **Deaktiviert** |
    | **Erweiterungen** | *Keine* |
    | **Tags** | *Keine* |
   
1. Warten Sie, bis die Bereitstellung abgeschlossen wurde. Dies sollte einige Minuten dauern.

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie Azure-Ressourcen bereitgestellt, die erforderlich sind, um hoch verfügbare SAP NetWeaver-Bereitstellungen zu unterstützen


## Übung 2: Konfigurieren des Betriebssystems von Azure-VMs unter Windows zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung.

Duration (Dauer): 60 Minuten

In dieser Übung konfigurieren Sie das Betriebssystem von Azure-VMs mit Windows Server, um eine hochverfügbare SAP NetWeaver-Bereitstellung zu ermöglichen.

### Aufgabe 1: Verknüpfen Sie Azure-VMs unter Windows Server 2016 mit der Active Directory-Domäne.

   > **Hinweis:** Bevor Sie diese Aufgabe starten, stellen Sie sicher, dass die in der vorherigen Übung initiierten Vorlagenbereitstellungen erfolgreich abgeschlossen wurden. 

1. Navigieren Sie im Azure-Portal zum Blatt des virtuellen Netzwerks mit dem Namen **adVNET**, das in der ersten Übung dieses Labs automatisch bereitgestellt wurde.

1. Zeigen Sie das Blatt **adVNET – DNS-Server** an, und beachten Sie, dass das virtuelle Netzwerk mit den privaten IP-Adressen konfiguriert ist, die den Domänencontrollern, die in der ersten Übung dieses Labs bereitgestellt wurden, als DNS-Server zugewiesen sind.

1. Starten Sie im Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die die Ressourcen enthält, die Sie im vorherigen Vorgang bereitgestellt haben:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Azure-VMs unter Windows Server, die Sie in der dritten Aufgabe der vorherigen Übung für die Active Directory-Domäne **adatum.com** bereitgestellt haben, zu verknüpfen (achten Sie darauf, den Platzhalter `<password>` durch das zuvor in dieser Übung angegebene Kennwort zu ersetzen):

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### Aufgabe 2: Überprüfen Sie die Speicherkonfiguration der Azure-VMs der Datenschicht.

1. Navigieren Sie auf dem Labcomputer im Azure-Portal zum Blatt **az12003b-vm0**.

1. Stellen Sie auf dem Blatt **az12003b-vm0** über Remotedesktop eine Verbindung mit der Azure-VM „az12003b-vm0“ her. Geben Sie bei entsprechender Aufforderung die folgenden Anmeldeinformationen ein:

    -   Anmelden als: **Schüler/Student**

    -   Kennwort: *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben*

1. Verwenden Sie in der RDP-Sitzung für „az12003b-vm0“ Remotedesktop, um eine Verbindung mit der Azure-VM **i20-db-0.adatum.com** herzustellen Geben Sie bei entsprechender Aufforderung die folgenden Anmeldeinformationen ein:

    -   Anmelden als: **ADATUM\\Kursteilnehmer**

    -   Kennwort: *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben*

1. Verwenden Sie Remotedesktop, um mit denselben Anmeldeinformationen eine Verbindung mit der Azure-VM **i20-db-1.adatum.com** herzustellen.

1. Verwenden Sie in der RDP-Sitzung für „i20-db-0.adatum.com“ die Datei- und Speicherdienste im Server-Manager, um die Datenträgerkonfiguration zu untersuchen. Beachten Sie, dass ein einzelner Datenträger über Volumeeinbindungen konfiguriert wurde, um Speicher für Datenbank- und Protokolldateien bereitzustellen. 

1. Verwenden Sie in der RDP-Sitzung für „i20-db-1.adatum.com“ die Datei- und Speicherdienste im Server-Manager, um die Datenträgerkonfiguration zu untersuchen. Beachten Sie, dass ein einzelner Datenträger über Volumeeinbindungen konfiguriert wurde, um Speicher für Datenbank- und Protokolldateien bereitzustellen. 


### Aufgabe 3: Bereiten Sie die Konfiguration des Failoverclusterings auf Azure-VMs mit Windows Server 2016 zur Unterstützung einer hoch verfügbaren SAP NetWeaver-Installation vor.

1. Starten Sie innerhalb der RDP-Sitzung für „i20-db-0.adatum.com“ eine Windows PowerShell ISE-Sitzung, und installieren Sie Failoverclustering- und Remoteverwaltungstools, indem Sie folgenden Code auf dem Paar der ASCS- und DB-Server ausführen, die zu Knoten der ASCS- und. SQL Server-Cluster werden:

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Hinweis:** Dies kann zu einem Neustart des Gastbetriebssystems aller vier Azure-VMs führen.

1. Klicken Sie auf dem Labcomputer im Azure-Portal auf **+ eine Ressource erstellen**.

1. Initiieren Sie auf dem Blatt **Neu** die Erstellung eines neuen **Speicherkontos** mit den folgenden Einstellungen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements* |
    | **Ressourcengruppe** | *der Name der Ressourcengruppe, in der Sie die Azure-VMs bereitgestellt haben, die eine hoch verfügbare SAP NetWeaver-Bereitstellung hosten* |
    | **Speicherkontoname** | *ein eindeutiger Name aus 3 bis 24 Buchstaben und Ziffern* |
    | **Location** | *dieselbe Azure-Region, in der Sie die Azure-VMs in der vorherigen Übung bereitgestellt haben* |
    | **Leistung** | **Standard** |
    | **Redundanz** | **Lokal redundanter Speicher (LRS)** |
    | **Konnektivitätsmethode** | **Öffentlicher Endpunkt (alle Netzwerke)** |
    | **Sichere Übertragung für REST-API-Vorgänge erforderlich** | **Enabled** |
    | **Große Dateifreigaben** | **Disabled** |
    | **Vorläufiges Löschen für Blobs, Container und Dateien** | **Disabled** |
    | **Hierarchischer Namespace** | **Disabled** |


### Aufgabe 4: Konfigurieren des Failoverclusterings auf Azure-VMs mit Windows Server 2016 zur Unterstützung einer hoch verfügbaren Datenbankebene der SAP NetWeaver-Installation.

1. Verwenden Sie ggf. in der RDP-Sitzung für „az12003b-vm0“ Remotedesktop, um die Verbindung mit der Azure-VM **i20-db-0.adatum.com** wiederherzustellen Geben Sie bei entsprechender Aufforderung die folgenden Anmeldeinformationen ein:

    -   Anmelden als: **ADATUM\\Kursteilnehmer**

    -   Kennwort: *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben*

1. Navigieren Sie in der RDP-Sitzung für „i20-db-0.adatum.com“ in Server-Manager zur Ansicht **Lokaler Server**, und deaktivieren Sie **Verstärkte Sicherheitskonfiguration für IE**.

1. Starten Sie in der RDP-Sitzung für „i20-db-0.adatum.com“ in Server-Manager über das Menü **Extras** das **Active Directory-Verwaltungscenter**.

1. Erstellen Sie im Active Directory-Verwaltungscenter eine neue Organisationseinheit namens **Cluster** im Stammverzeichnis der Domäne „adatum.com“.

1. Verschieben Sie im Active Directory-Verwaltungscenter die Computerkonten von „i20-db-0“ und „i20-db-1“ aus dem Container **Computer** in die Organisationseinheit **Cluster**.

1. Starten Sie in der RDP-Sitzung für „i20-db-0“ eine Windows PowerShell ISE-Sitzung, und erstellen Sie einen neuen Cluster, indem Sie folgenden Code ausführen:

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.15
    ```

1. Wechseln Sie innerhalb der RDP-Sitzung für „i20-db-0.adatum.com“ zur Konsole **Active Directory-Verwaltungscenter**.

1. Navigieren Sie im Active Directory-Verwaltungscenter zur Organisationseinheit **Cluster**, und zeigen Sie deren **Eigenschaftenfenster** an. 

1. Navigieren Sie im Fenster **Eigenschaften** der Organisationseinheit **Cluster** zum Abschnitt **Erweiterungen**, und öffnen Sie die Registerkarte **Sicherheit**. 

1. Klicken Sie auf der Registerkarte **Sicherheit** auf die Schaltfläche **Erweitert**, um das Fenster **Erweiterte Sicherheitseinstellungen für Cluster** zu öffnen. 

1. Klicken Sie auf der Registerkarte **Berechtigungen** des Fensters **Erweiterte Sicherheitseinstellungen für Cluster** auf **Hinzufügen**.

1. Klicken Sie im Fenster **Berechtigungseintrag für Cluster** auf **Prinzipal auswählen**.

1. Klicken Sie im Dialogfeld **Benutzer*in, Dienstkonto oder Gruppe auswählen** auf **Objekttypen**, aktivieren Sie das Kontrollkästchen neben dem Eintrag **Computer**, und klicken Sie auf **OK**. 

1. Geben Sie im Dialogfeld **Benutzer*in, Computer, Dienstkonto oder Gruppe auswählen** im Dialogfeld **Namen des auszuwählenden Objekts eingeben** **az12003b-db-cl0** ein, und klicken Sie auf **OK**.

1. Stellen Sie im Fenster **Berechtigungseintrag für Cluster** sicher, dass **Zulassen** in der Dropdownliste **Typ** angezeigt wird. Wählen Sie dann in der Dropdownliste **Gilt für** die Option **Dieses und alle untergeordneten Objekte** aus. Aktivieren Sie in der Liste **Berechtigungen** die Kontrollkästchen **Computerobjekte erstellen** und **Computerobjekte löschen**, und klicken Sie zweimal auf **OK**.

1. Installieren Sie in der Windows PowerShell ISE-Sitzung das PowerShell-Modul „Az“, indem Sie Folgendes ausführen:

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Authentifizieren Sie sich bei der Windows PowerShell ISE-Sitzung mithilfe Ihrer Azure AD-Anmeldeinformationen, indem Sie Folgendes ausführen:

    ```
    Add-AzAccount
    ```

    > **Hinweis:** Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Geschäfts-, Schul- oder Uni- oder persönlichen Microsoft-Konto mit der Besitzer- oder der Mitwirkendenrolle beim Azure-Abonnement an, das Sie für dieses Lab verwenden.

1. Führen Sie in der Windows PowerShell ISE-Sitzung den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die das Speicherkonto enthält, das Sie in der vorherigen Aufgabe bereitgestellt haben:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Führen Sie in der Windows PowerShell ISE-Sitzung folgenden Code aus, um das Cloudzeugenquorum des neuen Clusters festzulegen:

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Zur Überprüfung der resultierenden Konfiguration starten Sie in der RDP-Sitzung für „i20-db-0.adatum.com“ über das Menü **Tools** in Server-Manager den **Failovercluster-Manager**.

1. Überprüfen Sie in der Konsole des **Failovercluster-Managers** die Clusterkonfiguration **az12003b-db-cl0**, einschließlich der Knoten sowie der Zeugen- und Netzwerkeinstellungen. Beachten Sie, dass der Cluster keinen freigegebenen Speicher hat.


### Aufgabe 6: Konfigurieren des Failoverclusterings auf Azure-VMs mit Windows Server 2016 zur Unterstützung einer hoch verfügbaren ASCS-Ebene der SAP NetWeaver-Installation.

> **Hinweis:** Stellen Sie sicher, dass die Bereitstellung des S2D-Clusters, den Sie in Aufgabe 4 von Übung 1 initiiert haben, erfolgreich abgeschlossen wurde, bevor Sie diese Aufgabe starten.

1. Verwenden Sie in der RDP-Sitzung für „az12003b-vm0“ Remotedesktop, um eine Verbindung mit der Azure-VM **i20-ascs-0.adatum.com** herzustellen Geben Sie bei entsprechender Aufforderung die folgenden Anmeldeinformationen ein:

    -   Anmelden als: **ADATUM\\Kursteilnehmer**

    -   Kennwort: *dasselbe Kennwort, das Sie zuvor in diesem Lab angegeben haben*

1. Navigieren Sie in der RDP-Sitzung für „i20-ascs-0.adatum.com“ in Server-Manager zur Ansicht **Lokaler Server**, und deaktivieren Sie **Verstärkte Sicherheitskonfiguration für IE**.

1. Starten Sie in der RDP-Sitzung für „i20-ascs-0.adatum.com“ in Server-Manager über das Menü **Extras** das **Active Directory-Verwaltungscenter**.

1. Navigieren Sie im Active Directory-Verwaltungscenter zum Container **Computer**. 

1. Verschieben Sie im Active Directory-Verwaltungscenter die Computerkonten von „i20-ascs-0“ und „i20-ascs-1“ aus dem Container **Computer** in die Organisationseinheit **Cluster**.

1. Starten Sie in der RDP-Sitzung für „i20-ascs-0.adatum.com“ eine Windows PowerShell ISE-Sitzung, und erstellen Sie einen neuen Cluster, indem Sie folgenden Code ausführen:

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.16
    ```

1. Wechseln Sie innerhalb der RDP-Sitzung für „i20-ascs-0.adatum.com“ zur Konsole **Active Directory-Verwaltungscenter**.

1. Navigieren Sie im Active Directory-Verwaltungscenter zur Organisationseinheit **Cluster**, und zeigen Sie deren **Eigenschaftenfenster** an. 

1. Navigieren Sie im Fenster **Eigenschaften** der Organisationseinheit **Cluster** zum Abschnitt **Erweiterungen**, und öffnen Sie die Registerkarte **Sicherheit**. 

1. Klicken Sie auf der Registerkarte **Sicherheit** auf die Schaltfläche **Erweitert**, um das Fenster **Erweiterte Sicherheitseinstellungen für Cluster** zu öffnen. 

1. Klicken Sie auf der Registerkarte **Berechtigungen** des Fensters **Erweiterte Sicherheitseinstellungen für Computer** auf **Hinzufügen**.

1. Klicken Sie im Fenster **Berechtigungseintrag für Cluster** auf **Prinzipal auswählen**.

1. Klicken Sie im Dialogfeld **Benutzer*in, Dienstkonto oder Gruppe auswählen** auf **Objekttypen**, aktivieren Sie das Kontrollkästchen neben dem Eintrag **Computer**, und klicken Sie auf **OK**. 

1. Geben Sie im Dialogfeld **Benutzer*in, Computer, Dienstkonto oder Gruppe auswählen** im Dialogfeld **Namen des auszuwählenden Objekts eingeben** **az12003b-ascs-cl0** ein, und klicken Sie auf **OK**.

1. Stellen Sie im Fenster **Berechtigungseintrag für Cluster** sicher, dass **Zulassen** in der Dropdownliste **Typ** angezeigt wird. Wählen Sie dann in der Dropdownliste **Gilt für** die Option **Dieses und alle untergeordneten Objekte** aus. Aktivieren Sie in der Liste **Berechtigungen** die Kontrollkästchen **Computerobjekte erstellen** und **Computerobjekte löschen**, und klicken Sie zweimal auf **OK**.

1. Installieren Sie in der Windows PowerShell ISE-Sitzung das PowerShell-Modul „Az“, indem Sie Folgendes ausführen:

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Authentifizieren Sie sich bei der Windows PowerShell ISE-Sitzung mithilfe Ihrer Azure AD-Anmeldeinformationen, indem Sie Folgendes ausführen:

    ```
    Add-AzAccount
    ```

    > **Hinweis:** Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Geschäfts-, Schul- oder Uni- oder persönlichen Microsoft-Konto mit der Besitzer- oder der Mitwirkendenrolle beim Azure-Abonnement an, das Sie für dieses Lab verwenden.

1. Führen Sie in der Windows PowerShell ISE-Sitzung den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die das Speicherkonto enthält, das Sie weiter oben in dieser Übung bereitgestellt haben:

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Führen Sie in der Windows PowerShell ISE-Sitzung folgenden Code aus, um das Cloudzeugenquorum des Clusters festzulegen:

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Zur Überprüfung der resultierenden Konfiguration starten Sie in der RDP-Sitzung für „i20-ascs-0.adatum.com“ über das Menü **Tools** in Server-Manager den **Failovercluster-Manager**.

1. Überprüfen Sie in der Konsole des **Failovercluster-Managers** die Clusterkonfiguration **az12003b-ascs-cl0**, einschließlich der Knoten sowie der Zeugen- und Netzwerkeinstellungen. Beachten Sie, dass der Cluster keinen freigegebenen Speicher hat.


### Aufgabe 7: Festlegen von Berechtigungen für die Freigabe\\\\GLOBALHOST\\sapmnt

In dieser Aufgabe legen Sie Berechtigungen auf Freigabeebene für die Freigabe **\\\\GLOBALHOST\\sapmnt** fest.

> **Hinweis:** Standardmäßig werden die Vollzugriffsberechtigungen nur für das ADATUM\Student-Konto gewährt. 

1. Führen Sie in der Remotedesktopsitzung für „i20-ascs-0.adatum.com“ im Fenster **Windows PowerShell ISE** folgenden Code aus:

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### Aufgabe 8: Konfigurieren von Betriebssystemvoraussetzungen für die Installation von SAP NetWeaver ASCS und Datenbankkomponenten

1. Führen Sie in der Remotedesktopsitzung für „i20-ascs-0.adatum.com“ in der Windows PowerShell ISE-Sitzung folgenden Code aus, um Registrierungseinträge zu konfigurieren, die erforderlich sind, um die Installation von SAP ASCS-Komponenten und die Verwendung virtueller Namen zu erleichtern:

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie das Betriebssystem von Azure-VMs unter Windows konfiguriert, um eine hoch verfügbare SAP NetWeaver-Bereitstellung zu unterstützen


## Übung 3: Entfernen der Labressourcen

Dauer: 10 Minuten

In dieser Übung entfernen Sie die in diesem Lab bereitgestellten Ressourcen.

#### Aufgabe 1: Cloud Shell öffnen

1. Klicken Sie oben im Portal auf das **Cloud Shell**-Symbol, um den Cloud Shell-Bereich zu öffnen und wählen Sie PowerShell als Shell aus.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, das Paar von **Windows Server 2019 Datacenter**-Azure-VMs enthält, die Sie in der ersten Übung dieses Labs bereitgestellt haben:

    ```
    $resourceGroupNamePrefix = 'az12003b-'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um alle Ressourcengruppen auflisten zu können, die Sie in dieser Übung erstellt haben:

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. Vergewissern Sie sich, dass die Ausgabe nur die Ressourcengruppen enthält, die Sie in diesem Lab erstellt haben. Diese Gruppen werden in der nächsten Aufgabe gelöscht.

#### Aufgabe 2: Löschen der Ressourcengruppen

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um alle Ressourcengruppen zu löschen, die Sie in diesem Lab erstellt haben:

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. Schließen Sie den Cloud Shell-Bereich.

> **Result**: Durch den Abschluss dieser Übung haben Sie die in diesem Lab verwendeten Ressourcen entfernt.
