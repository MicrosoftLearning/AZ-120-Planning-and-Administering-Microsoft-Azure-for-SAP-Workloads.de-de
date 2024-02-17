---
lab:
  title: 04a – Implementieren der SAP-Architektur auf Azure-VMs unter Linux
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 Modul 4: Bereitstellen von SAP in Azure
# Lab 4a: Implementieren der SAP-Architektur auf Azure-VMs unter Linux

Geschätzte Dauer: 100 Minuten

Alle Aufgaben in diesem Lab werden über das Azure-Portal ausgeführt (einschließlich der Bash Cloud Shell-Sitzung)  

   > **Hinweis:** Wenn Sie Cloud Shell nicht verwenden, muss auf dem virtuellen Labcomputer die Azure CLI [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows) installiert sein.

Labdateien: keine

## Szenario
  
Als Vorbereitung für die Bereitstellung von SAP NetWeaver in Azure möchte die Adatum Corporation eine Demo implementieren, die die Hochverfügbarkeitsimplementierung von SAP NetWeaver auf Azure-VMs veranschaulicht, auf denen die SUSE-Distribution von Linux ausgeführt wird.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

-   Bereitstellen von Azure-Ressourcen, die zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung erforderlich sind

-   Konfigurieren des Betriebssystems von Azure-VMs unter Linux zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung

-   Konfigurieren des Clusterings von Azure-VMs unter Linux zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung

## Anforderungen

-   Ein Microsoft Azure-Abonnement mit der ausreichenden Anzahl verfügbarer Dsv3- und D2s-vCPUs (vier Standard_v3_D4s-VMs mit jeweils 2 vCPU und zwei Standard_v3_v3-VMs mit jeweils 4 vCPUs) in einer Azure-Region, die Verfügbarkeitszonen unterstützt

-   Ein Labcomputer mit einem Azure Cloud Shell-kompatiblem Webbrowser und Zugriff auf Azure


## Übung 1: Bereitstellen von Azure-Ressourcen, die zur Unterstützung von hochverfügbaren SAP NetWeaver-Bereitstellungen erforderlich sind

Dauer: 30 Minuten

In dieser Übung stellen Sie Azure-Infrastruktur-Computekomponenten bereit, die zum Konfigurieren von Linux-Clustering erforderlich sind. Dies umfasst das Erstellen eines Paars von Azure-VMs, auf denen Linux SUSE ausgeführt wird, in derselben Verfügbarkeitsgruppe.

### Aufgabe 1: Erstellen Sie ein virtuelles Netzwerk, das eine hoch verfügbare SAP NetWeaver-Bereitstellung hosten wird.

1. Starten Sie auf dem Laborcomputer einen Webbrowser, und navigieren Sie zum Azure-Portal unter **https://portal.azure.com**.

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Arbeits-, Schul- oder persönlichen Microsoft-Konto mit der Rolle „Besitzer“ oder „Mitwirkender“ beim Azure-Abonnement an, das Sie für dieses Lab verwenden, und mit der Rolle „Globaler Administrator“ im Azure AD-Mandanten, der Ihrem Abonnement zugeordnet ist.

1. Starten Sie im Azure-Portal eine Bash-Sitzung in Cloud Shell. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal im aktuellen Azure-Abonnement starten, werden Sie aufgefordert, eine Azure-Dateifreigabe zu erstellen, um Cloud Shell-Dateien zu speichern. Akzeptieren Sie in diesem Falls die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Azure-Region anzugeben, die Verfügbarkeitszonen unterstützt und wo Sie Ressourcen für dieses Lab erstellen möchten (ersetzen Sie `<region>` durch den Namen der Azure-Region, die Verfügbarkeitszonen unterstützt):

    ```cli
    LOCATION='<region>'
    ```

    > **Hinweis:** Erwägen der Verwendung der Regionen **USA, Osten** oder **USA, Osten 2** für die Bereitstellung Ihrer Ressourcen 

    > **Hinweis:** Stellen Sie sicher, dass die richtige Schreibweise für die Azure-Region verwendet wird (kurzer Name, der kein Leerzeichen enthält, z. B. **Eastus** anstelle von **USA, Osten**)

    > **Hinweis:** Informationen zum Identifizieren von Azure-Regionen, die Verfügbarkeitszonen unterstützen, finden Sie unter [https://docs.microsoft.com/en-us/azure/availability-zones/az-region](https://docs.microsoft.com/en-us/azure/availability-zones/az-region)

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `RESOURCE_GROUP_NAME` auf den Namen der Ressourcengruppe festzulegen, welche die Ressourcen enthält, die Sie in der vorherigen Aufgabe bereitgestellt haben:

    ```cli
    RESOURCE_GROUP_NAME='az12003a-sap-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um eine Ressourcengruppe in der von Ihnen angegebenen Region zu erstellen:

    ```cli
    az group create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um ein virtuelles Netzwerk mit einem einzelnen Subnetz in der von Ihnen erstellten Ressourcengruppe zu erstellen:

    ```cli
    VNET_NAME='az12003a-sap-vnet'

    VNET_PREFIX='10.3.0.0/16'

    SUBNET_NAME='sapSubnet'

    SUBNET_PREFIX='10.3.0.0/24'

    az network vnet create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION --name $VNET_NAME --address-prefixes $VNET_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX
    ```

1.  Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Ressourcen-ID des Subnetzes des neu erstellten virtuellen Netzwerks zu identifizieren und in der variablen SUBNET_ID zu speichern:

    ```cli
    SUBNET_ID=$(az network vnet subnet list --resource-group $RESOURCE_GROUP_NAME --vnet-name $VNET_NAME --query "[?name == '$SUBNET_NAME'].id" --output tsv)
    ```

### Aufgabe 2: Bereitstellen einer Bicep-Vorlage, die Azure-VMs mit Linux SUSE bereitstellt, die eine hoch verfügbare SAP NetWeaver-Bereitstellung hosten

1.  Führen Sie auf dem Labcomputer im Cloud Shell-Bereich die folgenden Befehle aus, um einen flachen Klon des Repositorys zu erstellen, das die Bicep-Vorlage hostet, die Sie für die Bereitstellung eines Azure-VM-Paars verwenden, das eine hoch verfügbare Installation von SAP HANA hostet und das aktuelle Verzeichnis auf den Speicherort dieser Vorlage und der zugehörigen Parameterdatei festgelegt:

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/sap/sap-3-tier-marketplace-image-md/
    ```

1.  Führen Sie im Cloud Shell-Bereich die folgenden Befehle aus, um den Namen des Administratorbenutzerkontos und dessen Kennwort festzulegen:

    ```
    ADMINUSERNAME='student'
    ADMINPASSWORD='Pa55w.rd1234'
    ```

1.  Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um Ressourcen zu identifizieren, die in der bevorstehenden Bereitstellung enthalten sein werden:

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group what-if --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters03a.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Überprüfen Sie die Ausgabe des Befehls, und stellen Sie sicher, dass sie keine Fehler enthält (Warnungen ignorieren). Führen Sie dann im Cloud Shell-Bereich den folgenden Befehl aus, um die Bereitstellung zu starten:

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group create --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Überprüfen Sie die Ausgabe des Befehls und stellen Sie sicher, dass er keine Fehler und Warnungen enthält. Wenn Sie dazu aufgefordert werden, drücken Sie die **Eingabetaste**, um mit der Bereitstellung fortzufahren.

1.  Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. 

    > **Hinweis:** Wenn die Bereitstellung während der Bereitstellung der CustomScriptExtension-Komponente mit der Fehlermeldung **Konflikt** fehlschlägt, führen Sie die folgenden Schritte aus, um dieses Problem zu beheben:

       - Überprüfen Sie im Azure-Portal auf dem **Bereitstellungsblatt** die Bereitstellungsdetails und identifizieren Sie die VM(en), auf denen die Installation der CustomScriptExtension fehlgeschlagen ist

       - Navigieren Sie im Azure-Portal zu dem Blatt der VM(en), die Sie im vorherigen Schritt identifiziert haben, wählen Sie **Erweiterungen** und aus dem Blatt **Erweiterungen** aus, entfernen Sie die CustomScript-Erweiterung

       - Führen Sie den vorherigen Schritt dieser Aufgabe erneut aus.

### Aufgabe 3: Bereitstellen eines Bastion Host

   > **Hinweis:** Da Azure-VMs, die Sie in der vorherigen Aufgabe bereitgestellt haben, über das Internet nicht zugänglich sind, stellen Sie einen virtuellen Azure-Computer mit Windows Server 2019 Datacenter bereit, der als Bastion Host fungiert. 

1. Klicken Sie auf dem Labcomputer im Azure-Portal auf **+ eine Ressource erstellen**.

1. Initiieren Sie auf dem Blatt **Neu** die Erstellung einer neuen Azure-VM basierend auf dem **Windows Server 2019 Datacenter**-Image.

1. Stellen Sie einen virtuellen Azure-Computer mit den folgenden Einstellungen bereit (behalten Sie alle anderen mit ihren Standardwerten bei):

    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Name einer neuen Ressourcengruppe* **az12003a-dmz-RG** |
    | **Name des virtuellen Computers** | **az12003a-vm0** |
    | **Region** | *dieselbe Azure-Region, in der Sie Azure-VMs in den vorherigen Aufgaben dieser Übung bereitgestellt haben* |
    | **Verfügbarkeitsoptionen** | **Keine Infrastrukturredundanz erforderlich** |
    | **Image** | *wählen Sie* **Windows Server 2019 Datacenter – Gen2** aus |
    | **Größe** | **Standard-D2s_v3** oder ähnlich |
    | **Benutzername** | **Kursteilnehmer** |
    | **Kennwort** | beliebiges komplexes Kennwort Ihrer Wahl |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Möchten Sie eine vorhandene Windows Server-Lizenz verwenden?** | **Nein** |
    | **Typ des Betriebssystemdatenträgers** | **HDD Standard** |
    | **Virtuelles Netzwerk** | **az12003a-sap-vnet** |
    | **Subnetzname** | *ein neues Subnetz mit dem Namen* **BastionSubnet** |
    | **Subnetzadressbereich** | **10.3.255.0/24** |
    | **Öffentliche IP-Adresse** | *eine neue IP-Adresse mit dem Namen* **az12003a-vm0-ip** |
    | **NIC-Netzwerksicherheitsgruppe** | **Grundlegend**  |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Ein** |
    | **Optionen für den Lastenausgleich** | **Keine** |
    | **Aktivieren einer systemseitig zugewiesenen verwalteten Identität** | **Deaktiviert** |
    | **Mit Azure AD anmelden** | **Deaktiviert** |
    | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
    | **Optionen zur Patchorchestrierung** | **Manuelle Updates** |
    | **Startdiagnose** | **Deaktivieren** |
    | **Diagnose des Gastbetriebssystems aktivieren** | **Deaktiviert** |
    | **Erweiterungen** | *Keine* |
    | **Tags** | *Keine* |

   > **Hinweis:** Stellen Sie sicher, dass Sie sich das Kennwort merken, das Sie während der Bereitstellung angegeben haben. Sie benötigen diese später in diesem Lab.

1. Warten Sie, bis die Bereitstellung abgeschlossen wurde. Dies sollte einige Minuten dauern.

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie Azure-Ressourcen bereitgestellt, die erforderlich sind, um hoch verfügbare SAP NetWeaver-Bereitstellungen zu unterstützen


## Übung 2: Konfigurieren von Azure-VMs unter Linux zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung.

Dauer: 30 Minuten

In dieser Übung konfigurieren Sie Azure-VMs, auf denen SUSE Linux Enterprise Server ausgeführt wird, um eine hoch verfügbare SAP NetWeaver-Bereitstellung aufzunehmen.

### Aufgabe 1: Konfigurieren sie das Netzwerk der Azure-VMs der Datenschicht.

   > **Hinweis:** Bevor Sie diese Aufgabe starten, stellen Sie sicher, dass die in der vorherigen Übung initiierten Vorlagenbereitstellungen erfolgreich abgeschlossen wurden. 

1. Navigieren Sie vom Labcomputer im Azure-Portal zum Blatt der Azure-VM **i20-db-0**.

1. Navigieren Sie vom Blatt **i20-db-0** zum **Netzwerk**-Blatt. 

1. Navigieren Sie vom **i20-db-0 – Netzwerk**-Blatt zur Netzwerkschnittstelle des i20-db-0. 

1. Navigieren Sie vom Blatt der Netzwerkschnittstelle des i20-db-0 zum Blatt der IP-Konfigurationen und zeigen Sie von dort aus das **Ipconfig1**-Blatt an.

1. Legen Sie auf dem Blatt **ipconfig1** die private IP-Adresse auf **10.3.0.20** fest, ändern Sie die Zuweisung in **Statisch** und speichern Sie die Änderung.

1. Navigieren Sie im Azure-Portal zum Blatt der Azure-VM **i20-db-1**.

1. Navigieren Sie vom Blatt **i20-db-1** zum **Netzwerk**-Blatt. 

1. Navigieren Sie vom Blatt **i20-db-1 – Netzwerk** zur Netzwerkschnittstelle des i20-db-1. 

1. Navigieren Sie vom Blatt der Netzwerkschnittstelle des i20-db-1 zum Blatt mit den IP-Konfigurationen und zeigen Sie von dort aus das **Ipconfig1**-Blatt an.

1. Legen Sie auf dem Blatt **ipconfig1** die private IP-Adresse auf **10.3.0.21** fest, ändern Sie die Zuweisung in **Statisch** und speichern Sie die Änderung.


### Aufgabe 2: Stellen Sie eine Verbindung mit den Azure-VMs der Datenschicht her.

1. Navigieren Sie vom Labcomputer im Azure-Portal zum Blatt **az12003a-vm0**.

1. Stellen Sie auf dem Blatt **az12003a-vm0** eine Verbindung mit der Azure-VM az12003a-vm0 über Remotedesktop her. Wenn Sie zur Authentifizierung aufgefordert werden, geben Sie den Benutzernamen und das Kennwort ein, das Sie während der Bereitstellung dieser VM festgelegt haben.

1. Navigieren Sie in der RDP-Sitzung zu az12003a-vm0 im Server-Manager zur Ansicht **Lokaler Server** und deaktivieren Sie die **erweiterte IE-Sicherheitskonfiguration**.

1. Laden Sie in der RDP-Sitzung auf az12003a-vm0 PuTTY herunter und installieren Sie es von [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

1. Verwenden Sie PuTTY, um sich über SSH mit **i20-db-0**-Azure-VM zu verbinden. Bestätigen Sie die Sicherheitswarnung und geben Sie bei Aufforderung die folgenden Anmeldeinformationen an:

    -   Anmelden als: **Schüler/Student**

    -   Kennwort: **Pa55w.rd1234**

1. Verwenden Sie PuTTY, um sich über SSH mit **i20-db-1**-Azure-VM mit denselben Anmeldeinformationen zu verbinden.


### Aufgabe 3: Überprüfen Sie die Speicherkonfiguration der Azure-VMs der Datenschicht.

1. Führen Sie in der PuTTY SSH-Sitzung zu i20-db-0 Azure-VM den folgenden Befehl aus, um die Berechtigungen zu erhöhen: 

    ```
    sudo su -
    ```

1. Wenn Sie zur Eingabe des Kennworts aufgefordert werden, geben Sie **Pa55w.rd1234** ein und drücken Sie die **Eingabetaste**. 

1. Überprüfen Sie in der SSH-Sitzung auf i20-db-0, ob alle SAP HANA-bezogenen Volumes (einschließlich **/usr/sap**, **/hana/shared**, **/hana/backup**, **/hana/data**und **/hana/logs**) ordnungsgemäß eingebunden werden, indem Sie Folgendes ausführen:

    ```
    df -h
    ```

1. Wiederholen Sie die vorherigen Schritte auf der Azure-VM i20-db-1.


### Aufgabe 4: Aktivieren des knotenübergreifenden Kennworts ohne SSH-Zugriff

1. Generieren Sie in der SSH-Sitzung zu i20-db-0 einen SSH-Schlüssel ohne Passphrase, indem Sie Folgendes ausführen:

    ```
    ssh-keygen
    ```

1. Wenn Sie dazu aufgefordert werden, drücken Sie dreimal die **Eingabetaste** und zeigen Sie dann den Schlüssel an, indem Sie Folgendes ausführen: 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Kopieren Sie den Wert des Schlüssels in die Zwischenablage.

1. Erstellen Sie in der SSH-Sitzung zu i20-db-1 die Datei **/root/.ssh/authorized\_-Schlüssel** im vi-Editor, indem Sie Folgendes ausführen:

    ```
    vi /root/.ssh/authorized_keys
    ```

1. Fügen Sie im vi-Editor den Schlüssel ein, den Sie auf i20-db-0 generiert haben.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Generieren Sie in der SSH-Sitzung zu i20-db-1 einen SSH-Schlüssel ohne Passphrase, indem Sie Folgendes ausführen:

    ```
    ssh-keygen
    ```

1. Wenn Sie dazu aufgefordert werden, drücken Sie dreimal die **Eingabetaste** und zeigen Sie dann den Schlüssel an, indem Sie Folgendes ausführen: 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Kopieren Sie den Wert des Schlüssels in die Zwischenablage.

1. Erstellen Sie in der SSH-Sitzung zu i20-db-0 die Datei **/root/.ssh/authorized\_-Schlüssel** im vi-Editor, indem Sie Folgendes ausführen:

    ```
    vi /root/.ssh/authorized_keys
    ```

1. Fügen Sie im vi-Editor den Schlüssel ein, den Sie auf i20-db-1 generiert haben, beginnend mit einer neuen Zeile.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Um zu überprüfen, ob die Konfiguration erfolgreich war, richten Sie in der SSH-Sitzung zu i20-db-0 eine SSH-Sitzung als **Stamm** von i20-db-0 zu i20-db-1 ein, indem Sie Folgendes ausführen: 

    ```
    ssh root@i20-db-1
    ```

1. Wenn Sie gefragt werden, ob Sie die Verbindung fortsetzen möchten, geben Sie `yes` ein und drücken Sie die **Eingabetaste**. 

1. Stellen Sie sicher, dass Sie nicht zur Eingabe des Kennworts aufgefordert werden.

1. Schließen Sie die SSH-Sitzung von i20-db-0 auf i20-db-1, indem Sie Folgendes ausführen: 

    ```
    exit
    ```

1. Richten Sie in der SSH-Sitzung zu i20-db-1 eine SSH-Sitzung als **Stamm** von i20-db-1 bis i20-db-0 ein, indem Sie Folgendes ausführen: 

    ```
    ssh root@i20-db-0
    ```

1. Wenn Sie gefragt werden, ob Sie die Verbindung fortsetzen möchten, geben Sie `yes` ein und drücken Sie die **Eingabetaste**. 

1. Stellen Sie sicher, dass Sie nicht zur Eingabe des Kennworts aufgefordert werden.

1. Schließen Sie die SSH-Sitzung von i20-db-1 zu i20-db-0, indem Sie Folgendes ausführen: 

    ```
    exit
    ```

### Aufgabe 5: Hinzufügen von YaST-Paketen, Aktualisieren des Linux-Betriebssystems und Installieren von HA-Erweiterungen

1. Führen Sie in der SSH-Sitzung zu i20-db-0 Folgendes aus, um YaST zu starten:

    ```
    yast
    ```

1. Wählen Sie im **YaST Control Center** die Option **Software\>-Add-On-Produkte** aus und drücken Sie auf **Eingabe**. Dadurch wird der **Paket-Manager** geladen.

1. Überprüfen Sie auf dem Bildschirm **Installierte Add-On-Produkte**, ob das **öffentliche Cloudmodul** bereits installiert ist. Drücken Sie dann zweimal **F9**, um zur Shell-Eingabeaufforderung zurückzukehren.

1. Führen Sie in der SSH-Sitzung zu i20-db-0 Folgendes aus, um das Betriebssystem zu aktualisieren (wenn Sie dazu aufgefordert werden, geben Sie **y** ein und drücken Sie die **Eingabetaste**):

    ```
    zypper update
    ```

1. Führen Sie in der SSH-Sitzung zu i20-db-0 Folgendes aus, um die für Clusterressourcen erforderlichen Pakete zu installieren (wenn Sie dazu aufgefordert werden, geben Sie **y** ein und drücken Sie die **Eingabetaste**):

    ```
    zypper in socat
    ```

1. Führen Sie in der SSH-Sitzung zu i20-db-0 Folgendes aus, um die Azure-lb-Komponente zu installieren, die von Clusterressourcen benötigt wird:

    ```
    zypper in resource-agents
    ```

1. Öffnen Sie in der SSH-Sitzung auf i20-db-0 die Datei **/etc/systemd/system.conf** im vi-Editor, indem Sie Folgendes ausführen:

    ```
    vi /etc/systemd/system.conf
    ```

1. Ersetzen Sie `#DefaultTasksMax=512` im Vi-Editor durch `DefaultTasksMax=4096`. 

    > **Hinweis:** In einigen Fällen kann Pacemaker viele Prozesse erstellen, die für ihre Anzahl auferlegte Standardgrenze erreichen und ein Failover auslösen. Diese Änderung erhöht die maximale Anzahl zulässiger Prozesse.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Führen Sie in der SSH-Sitzung zu i20-db-0 Folgendes aus, um die Konfigurationsänderung zu aktivieren:

    ```
    systemctl daemon-reload
    ```

1. Führen Sie in der SSH-Sitzung zu i20-db-0 Folgendes aus, um das Paket für Zaun-Agents zu installieren:

    ```
    zypper install fence-agents
    ```

1. Führen Sie in der SSH-Sitzung zu i20-db-0 den folgenden Befehl aus, um das für den Fence-Agenten erforderliche Azure Python SDK zu installieren (wenn Sie dazu aufgefordert werden, geben Sie **y** ein und drücken Sie die **Eingabetaste**):

    ```
    SUSEConnect -p sle-module-public-cloud/12/x86_64
    zypper install python-azure-mgmt-compute
    ```

1. Wiederholen Sie die vorherigen Schritte in diesem Vorgang auf i20-db-1.

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie das Betriebssystem von Azure-VMs unter Linux konfiguriert, um eine hoch verfügbare SAP NetWeaver-Bereitstellung zu unterstützen

## Übung 3: Konfigurieren des Clusterings von Azure-VMs unter Linux zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung.

Dauer: 30 Minuten

In dieser Übung konfigurieren Sie Clustering auf Azure VMs unter Linux, um eine hochverfügbare SAP NetWeaver-Bereitstellung zu unterstützen.

### Aufgabe 1: Konfigurieren von Clustering

1. Führen Sie in der RDP-Sitzung zu az12003a-vm0 in der PuTTY-basierten SSH-Sitzung zu i20-db-0 Folgendes aus, um die Konfiguration eines HA-Clusters auf i20-db-0 zu initiieren:

    ```
    ha-cluster-init -u
    ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Antworten:

    -   Möchten Sie trotzdem fortsetzen (y/n)?: **y**

    -   Adresse für Ring0 [10.3.0.20]: **EINGABETASTE**

    -   Port für Ring0 [5405]: **EINGABETASTE**

    -   Möchten Sie SBD verwenden (y/n)?: **n**

    -   Möchten Sie eine virtuelle IP-Adresse konfigurieren (y/n)?: **n**

    > **Hinweis:** Das Clustering-Setup generiert ein **Hacluster-Konto**, dessen Kennwort auf **Linux** festgelegt ist. Sie ändern es später in dieser Aufgabe.

1. Führen Sie in der RDP-Sitzung zu az12003a-vm0 in der PuTTY-basierten SSH-Sitzung zu i20-db-1 Folgendes aus, um dem HA-Cluster auf i20-db-0 von i20-db-1 beizutreten:

    ```
    ha-cluster-join
    ```

1. Wenn Sie dazu aufgefordert werden, geben Sie die folgenden Antworten:

    -   Möchten Sie dennoch fortsetzen (y/n)? **y**

    -   IP-Adresse oder Hostname vorhandener Knoten (z. B.: 192.168.1.1) \[\]: **i20-db-0**

    -   Adresse für Ring0 [10.3.0.21]: **EINGABETASTE**

1. Führen Sie in der PuTTY-basierten SSH-Sitzung auf i20-db-0 Folgendes aus, um das Kennwort des **hacluster**-Kontos auf **Pa55w.rd1234** festzulegen (geben Sie das neue Kennwort ein, wenn Sie dazu aufgefordert werden): 

    ```
    passwd hacluster
    ```

1. Wiederholen Sie den vorherigen Schritt auf i20-db-1.

### Aufgabe 2: Überprüfen der Corosync-Konfiguration

1. Öffnen Sie in der RDP-Sitzung auf az12003a-vm0 in der PuTTY-basierten SSH-Sitzung auf i20-db-0 die Datei **/etc/corosync/corosync.conf**, indem Sie Folgendes ausführen:

    ```
    vi /etc/corosync/corosync.conf
    ```

1. Beachten Sie im vi-Editor den `transport: udpu`-Eintrag und den `nodelist`-Abschnitt:
    ```
    [...]
       interface { 
           [...] 
       }
       transport:      udpu
    } 
    nodelist {
       node {
         ring0_addr:     10.3.0.20
         nodeid:     1
       }
       node {
         ring0_addr:     10.3.0.21
         nodeid:     2
       } 
    }
    logging {
        [...]
    ```

1. Ersetzen Sie im vi-Editor den Eintrag `token: 5000` durch `token: 30000`.

    > **Hinweis:** Diese Änderung ermöglicht die Erhaltung des Arbeitsspeichers. Weitere Informationen finden Sie in der [Microsoft-Dokumentation zur Wartung virtueller Computer in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-and-updates#maintenance-that-doesnt-require-a-reboot)

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Wiederholen Sie die vorherigen Schritte in i20-db-1.


### Aufgabe 3: Ermitteln des Werts der Azure-Abonnement-ID und der Azure AD-Mandanten-ID

1. Stellen Sie auf dem Labcomputer im Browserfenster im Azure-Portal unter **https://portal.azure.com** sicher, dass Sie mit dem Benutzerkonto angemeldet sind, das über die Rolle „Globaler Administrator“ im Azure AD-Mandanten verfügt, der Ihrem Abonnement zugeordnet ist.

1. Starten Sie im Azure-Portal eine Bash-Sitzung in Cloud Shell. 

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die ID Ihres Azure-Abonnements und die ID des entsprechenden Azure AD-Mandanten zu identifizieren:

    ```cli
    az account show --query '{id:id, tenantId:tenantId}' --output json
    ```

1. Kopieren Sie die resultierenden Werte in Editor. Sie werden dies in der nächsten Aufgabe benötigen.


### Aufgabe 4: Erstellen einer Azure AD-Anwendung für das STONITH-Gerät

1. Navigieren Sie im Azure-Portal zum Blatt Azure Active Directory.

1. Navigieren Sie auf dem Azure Active Directory-Blatt zum Blatt **App-Registrierungen** und klicken Sie dann auf **+ Neue Registrierung**:

1. Geben Sie auf dem Blatt **Anwendung registrieren** die folgenden Einstellungen an und klicken Sie auf **Registrieren**:

    -   Name: **Stonith-App**

    -   Unterstützter Kontotyp: **Nur Konten in diesem Organisationsverzeichnis**

1. Kopieren Sie auf dem Blatt **Stonith-App** den Wert der **Anwendungs-ID (Client-ID)** in Editor. Dies wird später in dieser Übung als **login_id** bezeichnet:

1. Klicken Sie auf dem Blatt **Stonith-App** auf **Zertifikate und Geheimnisse**.

1. Klicken Sie auf dem Blatt **Stonith-App – Zertifikate und Geheimnisse** auf **+ Neuer geheimer Clientschlüssel**.

1. Geben Sie im Bereich **Geheimen Clientschlüssel hinzufügen** in das Textfeld **Beschreibung** den **STONITH-App-Schlüssel** ein, lassen Sie im Abschnitt **Ablauf** den Standardwert ** Empfohlen stehen: 6 Monate** und klicken Sie dann auf **Hinzufügen**.

1. Kopieren Sie den resultierenden **Wert** in Editor (dieser Eintrag wird nur einmal angezeigt, nachdem Sie auf **Hinzufügen** geklickt haben). Dies wird später in dieser Übung als **Kennwort** bezeichnet:


### Aufgabe 5: Erteilen von Berechtigungen für Azure-VMs für den Dienstprinzipal der STONITH-App 

1. Navigieren Sie im Azure-Portal zum Blatt der Azure-VM **i20-db-0**

1. Zeigen Sie auf dem Blatt **i20-db-0** das Blatt **i20-db-0 – Zugriffssteuerung (IAM)** an.

1. Fügen Sie auf dem Blatt **i20-db-0 – Zugriffssteuerung (IAM)** eine Rollenzuweisung mit den folgenden Einstellungen hinzu:

    -   Rolle: **Mitwirkender von virtuellen Computern**

    -   Zugriff zuweisen an: **Azure AD-Benutzer, -Gruppe oder -Dienstprinzipal**

    -   Wählen Sie Folgendes: **Stonith-App**

1. Wiederholen Sie die vorherigen Schritte, um die Stonith-App der Rolle „Mitwirkender virtueller Computer“ der Azure-VM **i20-db-1** zuzuweisen


### Aufgabe 6: Konfigurieren des STONITH-Clustergeräts 

1. Wechseln Sie innerhalb der RDP-Sitzung zu az12003a-vm0 zur PuTTY-basierten SSH-Sitzung zu i20-db-0.

1. Führen Sie in der RDP-Sitzung zu az12003a-vm0 und in der PuTTY-basierten SSH-Sitzung zu i20-db-0 die folgenden Befehle aus (stellen Sie sicher, dass Sie die `subscription_id`, `tenant_id`, `login_id,` und `password`-Platzhalter durch die Werte ersetzen, die Sie in Übung 3 Aufgabe 4 identifiziert haben:

    ```
    crm configure property stonith-enabled=true

    crm configure property concurrent-fencing=true

    crm configure primitive rsc_st_azure stonith:fence_azure_arm \
      params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password" \
      pcmk_monitor_retries=4 pcmk_action_limit=3 power_timeout=240 pcmk_reboot_timeout=900 \
      op monitor interval=3600 timeout=120

    sudo crm configure property stonith-timeout=900
    ```

### Aufgabe 7: Überprüfen der Clusteringkonfiguration auf Azure-VMs, auf denen Linux mithilfe von Hawk ausgeführt wird

1. Starten Sie in der RDP-Sitzung auf az12003a-vm0 Internet Explorer und navigieren Sie zu **https://i20-db-0:7630**. Dadurch sollte die SUSE Hawk-Anmeldeseite angezeigt werden.

   > **Hinweis:** Ignorieren Sie ** Diese Seite ist keine sichere **-Meldung.

1. Melden Sie sich auf der SUSE Hawk-Anmeldeseite mit den folgenden Anmeldeinformationen an:

    -   Benutzername: **hacluster**

    -   Kennwort: **Pa55w.rd1234**

1. Stellen Sie sicher, dass der Clusterstatus fehlerfrei ist. Wenn eine Meldung angezeigt wird, die angibt, dass einer von zwei Clusterknoten nicht sauber ist, starten Sie diesen Knoten aus dem Azure-Portal neu.

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie Clustering auf Azure-VMs konfiguriert, auf denen Linux ausgeführt wird, um eine hoch verfügbare SAP NetWeaver-Bereitstellung zu unterstützen


## Übung 4: Entfernen der Labressourcen

Dauer: 10 Minuten

In dieser Übung entfernen Sie in diesem Lab bereitgestellte Ressourcen.

#### Aufgabe 1: Cloud Shell öffnen

1. Klicken Sie oben im Portal auf das **Cloud Shell**-Symbol, um den Cloud Shell-Bereich zu öffnen und wählen Sie Bash als Shell aus.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `RESOURCE_GROUP_PREFIX` auf das Präfix des Namens der Ressourcengruppe festzulegen, welche die in dieser Übung bereitgestellten Ressourcen enthält:

    ```cli
    RESOURCE_GROUP_PREFIX='az12003a-'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um alle Ressourcengruppen auflisten zu können, die Sie in dieser Übung erstellt haben:

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
    ```

1. Stellen Sie sicher, dass die Ausgabe nur die Ressourcengruppe enthält, die Sie in dieser Übung erstellt haben. Diese Ressourcengruppe mit allen Ressourcen wird in der nächsten Aufgabe gelöscht.

#### Aufgabe 2: Löschen der Ressourcengruppen

1. Führen Sie im Bereich Cloud Shell den folgenden Befehl aus, um die Ressourcengruppe und ihre Ressourcen zu löschen.

    ```cli
    az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

1. Schließen Sie den Cloud Shell-Bereich.

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie die in diesem Lab verwendeten Ressourcen entfernt.
