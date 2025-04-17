---
lab:
  title: 05 – Automatisieren der Bereitstellung mithilfe von Azure Center for SAP solutions
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Modul AZ 120: Entwerfen und Implementieren einer Infrastruktur zur Unterstützung von SAP-Workloads in Azure
# Lab: Automatisieren der Bereitstellung mithilfe von Azure Center for SAP solutions

Geschätzte Dauer: 100 Minuten

Alle Aufgaben in diesem Lab werden über das Azure-Portal ausgeführt.

## Ziele

In diesem Lab lernen Sie Folgendes:

- Implementieren von Voraussetzungen zum Bereitstellen von SAP-Workloads in Azure unter Verwendung von Azure Center for SAP solutions
- Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, unter Verwendung von Azure Center for SAP solutions

## Übung 1: Implementieren von Voraussetzungen zum Bereitstellen von SAP-Workloads in Azure unter Verwendung von Azure Center for SAP solutions

Duration (Dauer): 60 Minuten

In dieser Übung implementieren Sie die Voraussetzungen für die Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions Dazu gehören die folgenden Aufgaben:
- Erfüllen der vCPU-Anforderungen im Azure-Zielabonnement
- Konfigurieren der RBAC-Rollenzuweisungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird
- Erstellen eines Speicherkontos, das der für die Bereitstellung verwendeten Instanz von Azure Center for SAP solutions zugeordnet ist
- Erstellen einer benutzerseitig zugewiesenen verwalteten Identität, die von Azure Center for SAP solutions für die Authentifizierung und Autorisierung ihrer automatisierten Bereitstellung verwendet werden soll
- Erstellen einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die in Subnetzen des virtuellen Netzwerks verwendet werden soll, das die Bereitstellung hostet
- Erstellen von Routingtabellen, die in Subnetzen des virtuellen Netzwerks verwendet werden sollen, das die Bereitstellung hostet
- Erstellen und Konfigurieren des virtuellen Netzwerks, das die Bereitstellung hosten soll
- Bereitstellen von Azure Firewall im virtuellen Netzwerk, das die Bereitstellung hosten wird
- Bereitstellen von Azure Bastion im virtuellen Netzwerk, das die Bereitstellung hosten wird

Die Übung umfasst die folgenden Aufgaben:

- Aufgabe 1: Erfüllen der vCPU-Anforderungen im Azure-Zielabonnement
- Aufgabe 2: Konfigurieren der RBAC-Rollenzuweisungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird
- Aufgabe 3: Erstellen eines Speicherkontos, das der für die Bereitstellung verwendeten Instanz von Azure Center for SAP solutions zugeordnet ist
- Aufgabe 4: Erstellen und Konfigurieren einer benutzerseitig zugewiesenen verwalteten Identität, die von Azure Center for SAP solutions für die Authentifizierung und Autorisierung der automatisierten Bereitstellung verwendet werden soll
- Aufgabe 5: Erstellen einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die in Subnetzen des virtuellen Netzwerks verwendet werden soll, das die Bereitstellung hostet
- Aufgabe 6: Erstellen von Routingtabellen, die in Subnetzen des virtuellen Netzwerks verwendet werden sollen, das die Bereitstellung hostet
- Aufgabe 7: Erstellen und Konfigurieren des virtuellen Netzwerks, das die Bereitstellung hostet
- Aufgabe 8: Bereitstellen von Azure Firewall in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird
- Aufgabe 9: Bereitstellen von Azure Bastion in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird

### Aufgabe 1: Erfüllen der vCPU-Anforderungen im Azure-Zielabonnement

>**Hinweis:** Sie müssen diese Aufgabe nicht abschließen, wenn Sie bereits alle [Voraussetzungen des Labs Az-120](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab00_Prerequisites.md) implementiert haben.

>**Hinweis:** Um dieses Lab (wie beschrieben) abzuschließen, benötigen Sie ein Microsoft Azure-Abonnement mit den vCPU-Kontingenten, welche die Bereitstellung der folgenden VMs ermöglichen:

- 2 VMs vom Typ „Standard_E4ds_v4“ (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher) oder 2 VMs vom Typ „Standard_D4ds_v4“ (jeweils 4 vCPUs und 16 GiB Arbeitsspeicher) für die ASCS-Ebene
- 2 VMs vom Typ „Standard_E4ds_v4“ (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher) oder 2 VMs vom Typ „Standard_D4ds_v4“ (jeweils 4 vCPUs und 16 GiB Arbeitsspeicher) für die Logikschicht 
- 2 VMs vom Typ „Standard_M64ms“ (jeweils 64 vCPUs und 1750 GiB Arbeitsspeicher) für die Datenbankebene

>**Hinweis:** Um die vCPU- und Speicheranforderungen für die Datenbank-VMs zu minimieren, können Sie die VM-SKU in „Standard_M32ts“ ändern (jeweils 32 vCPUs und 192 GiB Arbeitsspeicher).

1. Starten Sie auf dem Labcomputer einen Webbrowser und navigieren Sie zum Azure-Portal unter `https://portal.azure.com`.
1. Wählen Sie im Azure-Portal das Symbol für **Cloud Shell** aus, und starten Sie eine PowerShell-Sitzung in Cloud Shell. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal in dem Azure-Abonnement starten, das Sie in diesem Lab verwenden, werden Sie aufgefordert, eine Azure-Dateifreigabe zum Speichern von Cloud Shell-Dateien zu erstellen. Akzeptieren Sie in diesem Fall die Standardwerte. Daraufhin wird ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt.

1. Führen Sie im Azure-Portal im Bereich **Cloud Shell** an der PowerShell-Eingabeaufforderung Folgendes aus (ersetzen Sie ggf. `eastus` durch den Namen der Azure-Region, in der Sie Ressourcen in diesem Lab bereitstellen möchten):

    > **Hinweis:** Um die Namen der Azure-Regionen zu ermitteln, führen Sie in **Cloud Shell** an der Bash-Eingabeaufforderung `(Get-AzLocation).Location` aus.
     
    ```powershell
    Set-Variable -Name "Azure_region" -Value ('eastus') -Option constant -Scope global -Description "All processes" -PassThru

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}
    
    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location $Azure_region | Where-Object {$_.Name.Value -eq 'cores'}
    ```

1. Überprüfen Sie die Ausgabe, um die aktuelle vCPU-Nutzung und den vCPU-Grenzwert zu identifizieren. Stellen Sie sicher, dass der Unterschied zwischen ihnen ausreicht, um vCPUs von Azure-VMs aufzunehmen, die Sie in diesem Lab bereitstellen. Berücksichtigen Sie sowohl die für die VM-Familie spezifische Zahl als auch die Zahl für die gesamten regionalen vCPUs. 
1. Wenn die Anzahl der vCPUs nicht ausreicht, schließen Sie den Cloud Shell-Bereich im Azure-Portal, suchen Sie im Textfeld **Suchen** nach **Kontingente**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Kontingente** die Option **Compute** aus.
1. Verwenden Sie auf der Seite **Kontingente \| Compute** den Filter **Region**, um die Azure-Region auszuwählen, in der Sie Ressourcen in diesem Lab bereitstellen möchten.
1. Suchen Sie in der Spalte **Kontingentname** den VM-SKU-Namen, für den eine Kontingenterhöhung erforderlich ist, und wählen Sie ihn aus. 
1. Überprüfen Sie in derselben Zeile den Eintrag in der Spalte **Anpassbar**. Der nächste Schritt hängt davon ab, ob die Spalte den Eintrag **Ja** oder **Nein** enthält.

   - Wenn der Eintrag auf **Ja** festgelegt ist, wählen Sie das Symbol **Anpassung anfordern** aus. Geben Sie unter **Neue Kontingentanforderung** im Textfeld **Neue Kontingentgrenze** die neue Kontingentgrenze ein, und wählen Sie dann **Absenden** aus.
   - Wenn der Eintrag auf **Nein** festgelegt ist, wählen Sie das Symbol **Zugriff anfordern oder Empfehlungen erhalten** und dann im Bereich **Kontingentempfehlungen** die Option **Support kontaktieren** und dann **Weiter** aus. 
1. Geben Sie auf der Registerkarte **Problembeschreibung** der Seite **Neue Supportanfrage** die folgenden Einstellungen an, und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Worauf bezieht sich Ihr Problem?|**Azure-Dienste**|
    |Problemtyp|**Grenzwerte für Dienste und Abonnements (Kontingente)**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Kontingenttyp|**Abonnementgrenzen für Compute/VM (Kerne/vCPUs) erhöhen**|

1. Wählen Sie auf der Registerkarte **Weitere Details** die Option **Details eingeben** aus.
1. Wählen Sie auf der Registerkarte **Kontingentdetails** in der Dropdownliste **Bereitstellungsmodell** die Option **Ressourcen-Manager** aus. Wählen Sie in der Dropdownliste **Standorte** die Azure-Zielregion aus, und wählen Sie in der Dropdownliste **Kontingente** die Azure-VM-Serie aus, für die Sie die Kontingentgrenzen erhöhen müssen. Geben Sie im Textfeld **Neue Grenze** die neue Kontingentgrenze ein, und wählen Sie dann **Speichern und fortfahren** aus.
1. Navigieren Sie zurück zur Registerkarte **Weitere Details**, und wählen Sie auf der Registerkarte **Erweiterte Diagnoseinformationen** die Option **Ja (Empfohlen)** aus.
1. Wählen Sie im Abschnitt **Supportmethode** entweder **E-Mail** oder **Telefon** als bevorzugte Kontaktmethode aus, und wählen Sie dann **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.

    > **Hinweis:** Warten Sie, bis die Anforderung zum Erhöhen der Kontingentgrenzen erfolgreich abgeschlossen wurde, bevor Sie mit der nächsten Aufgabe fortfahren.

### Aufgabe 2: Konfigurieren der RBAC-Rollenzuweisungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird

1. Starten Sie auf dem Labcomputer Microsoft Edge, und navigieren Sie zum Azure-Portal unter `https://portal.azure.com`.
1. Wenn Sie zur Authentifizierung aufgefordert werden, melden Sie sich mit den Microsoft Entra ID-Anmeldeinformationen mit der Rolle „Besitzer“ im Azure-Abonnement an, das Sie für dieses Lab verwenden. 
1. Suchen Sie im Azure-Portal im Textfeld **Suchen** nach **Abonnements**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Abonnements** den Eintrag aus, der das Azure-Abonnement darstellt, das Sie für dieses Lab verwenden werden. 
1. Wählen Sie auf der Seite, auf der die Eigenschaften des Azure-Abonnements angezeigt werden, die Option **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf der Seite **Zugriffssteuerung (IAM)** die Option **+ Hinzufügen** und dann im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Liste der **Auftragsfunktionsrollen** nach dem Eintrag **Azure Center for SAP solutions-Administrator**, und wählen Sie die entsprechende Option aus. Wählen Sie anschließend **Weiter** aus.
1. Klicken Sie auf der Registerkarte **Mitglieder** der Seite **Rollenzuweisung hinzufügen** auf **+ Mitglieder auswählen**. 
1. Geben Sie im Bereich **Mitglieder auswählen** im Textfeld **Auswählen** den Namen des Microsoft Entra ID-Benutzerkontos ein, das Sie für den Zugriff auf das Azure-Abonnement verwendet haben, das Sie für dieses Lab verwenden, wählen Sie es in der Liste der Ergebnisse aus, die Ihrem Eintrag entsprechen, und klicken Sie dann auf **Auswählen**.
1. Navigieren Sie zurück zur Registerkarte **Mitglieder**, und wählen Sie **Überprüfen + zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.
1. Wiederholen Sie die vorherigen sechs Schritte, um die Rolle **Operator für verwaltete Identität** dem Benutzerkonto zuzuweisen, das Sie für dieses Lab verwenden.

### Aufgabe 3: Erstellen eines Speicherkontos, das der für die Bereitstellung verwendeten Instanz von Azure Center for SAP solutions zugeordnet ist

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Speicherkonten**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Speicherkonten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Seite **Speicherkonto erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Erweitert >**.

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|der Name einer **neuen** Ressourcengruppe: **ACSS-DEMO**|
    |Speicherkontoname|Ein beliebiger global eindeutiger Name, der zwischen 3 und 24 Zeichen lang ist und aus Buchstaben und Ziffern besteht.|
    |Region|der Name der Azure-Region, in der Sie über ausreichende vCPU-Kontingente zum Ausführen dieses Labs verfügen|
    |Leistung|**Standard**|
    |Redundanz|**Georedundanter Speicher (GRS)**|
    |Bei regionaler Verfügbarkeit Lesezugriff auf die Daten bereitstellen|Deaktiviert|

1. Überprüfen Sie auf der Registerkarte **Erweitert** die verfügbaren Optionen, übernehmen Sie die Standardwerte, und wählen Sie **Weiter: Netzwerk >** aus.
1. Überprüfen Sie auf der Registerkarte **Netzwerk** die verfügbaren Optionen, stellen Sie sicher, dass die Option **Öffentlichen Zugriff über alle Netzwerke aktivieren** aktiviert ist, und wählen Sie **Überprüfen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie nicht, bis die Bereitstellung des Azure Storage-Kontos abgeschlossen ist. Fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefähr 2 Minuten dauern.

### Aufgabe 4: Erstellen und Konfigurieren einer benutzerseitig zugewiesenen verwalteten Identität, die von Azure Center for SAP solutions für die Authentifizierung und Autorisierung der automatisierten Bereitstellung verwendet werden soll

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Verwaltete Identitäten**, und wählen Sie das entsprechende Ergebnis aus.
1. Wählen Sie auf der Seite **Verwaltete Identitäten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Benutzerseitig zugewiesene verwaltete Identität erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**ACSS-DEMO**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab das Speicherkonto bereitgestellt haben|
    |Name|**Contoso-MSI**|

1. Warten Sie auf der Registerkarte **Überprüfen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie, bis die Bereitstellung der benutzerseitig zugewiesenen verwalteten Identität abgeschlossen ist. Dies sollte nur ein paar Sekunden dauern.

1. Navigieren Sie im Azure-Portal zur Seite **Verwaltete Identitäten**, und wählen Sie den Eintrag **Contoso-MSI** aus.
1. Wählen Sie auf der Seite **Contoso-MSI** die Option **Azure-Rollenzuweisungen** aus.
1. Wählen Sie auf der Seite **Azure-Rollenzuweisungen** die Option **+ Rollenzuweisung hinzufügen (Vorschau)** aus.
1. Geben Sie auf der Seite **+ Rollenzuweisung hinzufügen (Vorschau)** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Umfang|**Abonnement**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Rolle|**Azure Center for SAP solutions-Dienstrolle**|

2. Navigieren Sie zurück zur Seite **Azure-Rollenzuweisungen**, und wählen Sie **+ Rollenzuweisung hinzufügen (Vorschau)** aus.
3. Geben Sie auf der Seite **+ Rollenzuweisung hinzufügen (Vorschau)** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Umfang|**Storage**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Ressource|Der Name des Azure Storage-Kontos, das Sie in der vorherigen Aufgabe erstellt haben|
    |Rolle|**Lese- und Datenzugriff**|

### Aufgabe 5: Erstellen einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die in Subnetzen des virtuellen Netzwerks verwendet werden soll, das die Bereitstellung hostet

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Netzwerksicherheitsgruppen**, und wählen Sie das entsprechende Ergebnis aus.
1. Wählen Sie auf der Seite **Netzwerksicherheitsgruppen** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Netzwerksicherheitsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer **neuen** Ressourcengruppe: **CONTOSO-VNET-RG**|
    |Name|**ACSS-DEMO-NSG**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab das Speicherkonto bereitgestellt haben|

1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Standardmäßig ermöglichen die integrierten Regeln von Netzwerksicherheitsgruppen den gesamten ausgehenden Datenverkehr, den gesamten Datenverkehr innerhalb desselben virtuellen Netzwerks sowie den gesamten Datenverkehr zwischen virtuellen Netzwerken mit Peering. Dies reicht aus, um das Lab erfolgreich abzuschließen. Je nach Ihren Sicherheitsanforderungen sollten Sie möglicherweise einen Teil dieses Datenverkehrs blockieren. Wenn das der Fall ist, lesen Sie die Anleitungen in der Microsoft Learn-Dokumentation [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network).

### Aufgabe 6: Erstellen von Routingtabellen, die in Subnetzen des virtuellen Netzwerks verwendet werden sollen, das die Bereitstellung hostet

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Routingtabellen**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Routingtabellen** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Routingtabelle erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|
    |Name|**ACSS-ROUTE**|
    |Gatewayrouten verteilen|**Nein**|

1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

### Aufgabe 7: Erstellen und Konfigurieren des virtuellen Netzwerks, das die Bereitstellung hostet

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Virtuelle Netzwerke**, und wählen Sie die entsprechende Option aus. 
1. Wählen Sie auf der Seite **Virtuelle Netzwerke** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtuelles Netzwerk erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name des virtuellen Netzwerks|**CONTOSO-VNET**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|

1. Übernehmen Sie auf der Registerkarte **Sicherheit** die Standardeinstellungen, und wählen Sie **Weiter** aus.

    >**Hinweis:** Sie können zu diesem Zeitpunkt sowohl Azure Bastion als auch Azure Firewall bereitstellen, Sie stellen sie jedoch nach der Erstellung des virtuellen Netzwerks separat bereit.

1. Geben Sie auf der Registerkarte **IP-Adressen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |IP-Adressbereich|**/10.5.0.0/16 (65.536 Adressen)**|

    >**Hinweis:** Löschen Sie alle zuvor erstellten Subnetzeinträge. Sie fügen Subnetze hinzu, nachdem das virtuelle Netzwerk erstellt wurde.

1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.
1. Navigieren Sie zurück zur Seite **Virtuelle Netzwerke**, und wählen Sie den Eintrag **CONTOSO-VNET** aus. 
1. Wählen Sie auf der Seite **CONTOSO-VNET** in der vertikalen Menüleiste auf der linken Seite **Subnetze** aus.
1. Wählen Sie auf der Seite **CONTOSO-VNET \| Subnetze** die Option **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**app**|
    |Subnetzadressbereich|**10.5.0.0/24**|
    |Netzwerksicherheitsgruppe|**ACSS-DEMO-NSG**|
    |Routingtabelle|**ACSS-ROUTE**|

1. Navigieren Sie zurück zur Seite **CONTOSO-VNET \| Subnetze**, und wählen Sie **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**AzureBastionSubnet**|
    |Subnetzadressbereich|**10.5.1.0/26**|

1. Navigieren Sie zurück zur Seite **CONTOSO-VNET \| Subnetze**, und wählen Sie **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**db**|
    |Subnetzadressbereich|**10.5.2.0/24**|
    |Netzwerksicherheitsgruppe|**ACSS-DEMO-NSG**|
    |Routingtabelle|**ACSS-ROUTE**|

1. Navigieren Sie zurück zur Seite **CONTOSO-VNET \| Subnetze**, und wählen Sie **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz hinzufügen** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**AzureFirewallSubnet**|
    |Subnetzadressbereich|**10.5.3.0/24**|

### Aufgabe 8: Bereitstellen von Azure Firewall in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird

>**Hinweis:** Bevor Sie eine Azure Firewall-Instanz bereitstellen, erstellen Sie zuerst eine Firewallrichtlinie und eine öffentliche IP-Adresse, die von der Instanz verwendet werden soll.

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Firewallrichtlinien**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Firewallrichtlinien** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Azure Firewall-Richtlinie erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: DNS-Einstellungen >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**FirewallPolicy_contoso-firewall**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|
    |Richtlinientarif|**Standard**|
    |Übergeordnete Richtlinie|**Keine**|

1. Übernehmen Sie auf der Registerkarte **DNS-Einstellungen** die Standardoption **Deaktiviert**, und wählen Sie **Weiter: TLS-Inspektion >** aus.
1. Wählen Sie auf der Registerkarte **TLS-Inspektion** die Option **Weiter: Regeln >** aus.
1. Wählen Sie auf der Registerkarte **Regeln** die Option **+ Regelsammlung hinzufügen** aus.
1. Geben Sie im Bereich **Regelsammlung hinzufügen** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |Name|**AllowOutbound**|
    |Regelsammlungstyp|**Netzwerk**|
    |Priority|**101**|
    |Regelsammlungsaktion|**Zulassen**|
    |Regelsammlungsgruppe|**DefaultNetworkRuleCollectionGroup**|

1. Fügen Sie im Bereich **Regelsammlung hinzufügen** im Abschnitt **Regeln** eine Regel mit den folgenden Einstellungen hinzu:

    |Einstellung|Wert|
    |---|---|
    |Name|**RHEL**|
    |Quellentyp|**IP-Adresse**|
    |Quelle|*|
    |Protokoll|**Beliebige**|
    |Zielports|*|
    |Zieltyp|**IP-Adresse**|
    |Destination|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|

    >**Hinweis:** Informationen zur Ermittlung der für RHEL zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network).

    |Einstellung|Wert|
    |---|---|
    |Name|**ServiceTags**|
    |Quellentyp|**IP-Adresse**|
    |Quelle|*|
    |Protokoll|**Beliebige**|
    |Zielports|*|
    |Zieltyp|**Diensttag**|
    |Destination|**AzureActiveDirectory,AzureKeyVault,Storage**|

    >**Hinweis:** Bei Bedarf können Diensttags mit regionalen Bereichen verwendet werden. 

    |Einstellung|Wert|
    |---|---|
    |Name|**SUSE**|
    |Quellentyp|**IP-Adresse**|
    |Quelle|*|
    |Protokoll|**Beliebige**|
    |Zielports|*|
    |Zieltyp|**IP-Adresse**|
    |Destination|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|

    >**Hinweis:** Informationen zur Ermittlung der für SUSE zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network).

    |Einstellung|Wert|
    |---|---|
    |Name|**AllowOutbound**|
    |Quellentyp|**IP-Adresse**|
    |Quelle|*|
    |Protokoll|**TCP, UDP, ICMP, Beliebig**|
    |Zielports|*|
    |Zieltyp|**IP-Adresse**|
    |Destination|*|

1. Wählen Sie die Schaltfläche **Hinzufügen** aus, um alle Regeln zu speichern.
1. Navigieren Sie zurück zur Registerkarte **Regeln**, und wählen Sie **Weiter: IDPS >** aus.
1. Wählen Sie auf der Registerkarte **IDPS** die Option **Weiter: Threat Intelligence >** aus.

    >**Hinweis:** Für die IDPS-Funktionalität ist die Premium-SKU erforderlich.

1. Überprüfen Sie auf der Registerkarte **Threat Intelligence** die verfügbaren Einstellungen, ohne Änderungen vorzunehmen, und wählen Sie dann **Überprüfen + Erstellen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie, bis die Bereitstellung der Firewallrichtlinie abgeschlossen ist. Die Bereitstellung sollte ungefähr 1 Minuten dauern.

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Öffentliche IP-Adressen**, und wählen Sie das entsprechende Ergebnis aus.
1. Wählen Sie auf der Seite **Öffentliche IP-Adressen** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen einer öffentlichen IP-Adresse** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|
    |Name|**contoso-firewal-pip**|
    |IP-Version|**IPv4**|
    |SKU|**Standard**|
    |Verfügbarkeitszone|**Keine Zone**|
    |Tarif|**Regional**|
    |Routingpräferenz|**Microsoft Network**|
    |Leerlaufzeitüberschreitung (Minuten)|**4**|
    |DNS-Namensbezeichnung|nicht festgelegt|

1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie, bis die Bereitstellung der öffentlichen IP-Adresse abgeschlossen ist. Die Bereitstellung sollte einige Sekunden dauern.

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Firewalls**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Firewalls** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Firewall erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**contoso-firewall**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|
    |Verfügbarkeitszone|**Keine**|
    |Firewall-SKU|**Standard**|
    |Firewallverwaltung|**Firewallrichtlinie zum Verwalten dieser Firewall verwenden**|
    |Firewallrichtlinie|**FirewallPolicy_contoso-firewall**|
    |Virtuelles Netzwerk auswählen|**Vorhandenes verwenden**|
    |Virtuelles Netzwerk|**CONTOSO-VNET**|
    |Öffentliche IP-Adresse|**contoso-firewall-pip**|
    |Tunnelerzwingung|**Disabled**|

    >**Hinweis:** Warten Sie, bis die Bereitstellung von Azure Firewall abgeschlossen ist. Die Bereitstellung kann ungefähr 3 Minuten dauern.

1. Navigieren Sie im Azure-Portal zurück zur Seite **Firewalls**.
1. Wählen Sie auf der Seite **Firewalls** den Eintrag **contoso-firewall** aus.
1. Auf der Seite **contoso-firewall** sehen Sie, dass **Private IP-Adresse** auf **10.5.3.4** festgelegt ist. Dies ist die private IP-Adresse der Azure Firewall-Instanz.

    >**Hinweis:** Damit der Netzwerkdatenverkehr über Azure Firewall weitergeleitet werden kann, müssen Sie benutzerdefinierte Routen zu den Routingtabellen hinzufügen, die den App- und Datenbanksubnetzen des virtuellen Netzwerks zugeordnet sind, welche die SAP-Bereitstellung hosten.

1. Suchen Sie im Azure-Portal im Textfeld **Suchen** nach **Routingtabellen**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Routingtabellen** den Eintrag **ACSS-ROUTE** aus.
1. Wählen Sie auf der Seite **ACSS-ROUTE** die Option **Routen** aus.
1. Wählen Sie auf der Seite **ACSS-ROUTE \| Routen** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Route hinzufügen** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus:

    |Einstellung|Wert|
    |---|---|
    |Routenname|**Firewall**|
    |Zieltyp|**IP-Adressen**|
    |Ziel-IP-Adressen/CIDR-Bereiche|**0.0.0.0/0**|
    |Typ des nächsten Hops|**Virtuelles Gerät**|
    |Adresse des nächsten Hops|**10.5.3.4**|

### Aufgabe 9: Bereitstellen von Azure Bastion in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Bastion-Instanzen**, und wählen Sie die entsprechende Option aus. 
1. Wählen Sie auf der Seite **Bastion-Instanzen** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Bastion-Instanzen** die folgenden Einstellungen an, und wählen Sie **Weiter: Tags >** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**ACSS-BASTION**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|
    |Tarif|**Grundlegend**|
    |Anzahl von Instanzen|**2**|
    |Virtuelles Netzwerk|**CONTOSO-VNET**|
    |Subnetz|**AzureBastionSubnet**|
    |Öffentliche IP-Adresse|**Neu erstellen**|
    |Name der öffentlichen IP-Adresse|**ACSS-BASTION-PIP**|

1. Wählen Sie auf der Registerkarte **Tags** die Option **Weiter: Erweitert >**.
1. Überprüfen Sie auf der Registerkarte **Erweitert** die verfügbaren Einstellungen, ohne Änderungen vorzunehmen, und wählen Sie dann **Weiter: Überprüfen und erstellen >**.
1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie nicht, bis die Bereitstellung des Bastionhosts abgeschlossen ist. Fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefähr 15 Minuten dauern.

## Übung 2: Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, unter Verwendung von Azure Center for SAP solutions

Duration (Dauer): 40 Minuten

In dieser Übung verwenden Sie Azure Center for SAP solutions, um die Infrastruktur bereitzustellen, die SAP-Workloads in dem Azure-Abonnement hosten wird, das Sie in der vorherigen Übung verwendet haben. Nach der erfolgreichen Bereitstellung können Sie entweder mit der Installation von SAP-Software fortfahren, indem Sie Azure Center for SAP solutions verwenden oder die in diesem Lab bereitgestellten Azure-Ressourcen löschen.

>**Hinweis:** Informationen zum Installieren von SAP-Software mithilfe von Azure Center for SAP solutions finden Sie in der Microsoft Learn-Dokumentation, in der beschrieben wird, wie Sie [SAP-Installationsmedien abrufen](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) und [SAP-Software installieren](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software). Anweisungen zum Löschen der in diesem Lab bereitgestellten Azure-Ressourcen sind in der zweiten Aufgabe dieser Übung enthalten.

Die Übung besteht aus der folgenden Aufgabe:

- Aufgabe 1: Erstellen einer Instanz von Virtual Instance for SAP solutions
- Aufgabe 2: Löschen der in diesem Lab bereitgestellten Azure-Ressourcen

### Aufgabe 1: Erstellen einer Instanz von Virtual Instance for SAP solutions

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Azure Center for SAP solutions**, und wählen Sie die entsprechende Option aus. 
1. Wählen Sie unter **Azure Center for SAP solutions \| Übersicht** die Option **Erstellen eines neuen SAP-Systems** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtual Instance for SAP solutions erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: VMs** aus.

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer **neuen** Ressourcengruppe: **Contoso-SAP-C1S**|
    |Name (SID)|**C1S**|
    |Region|der Name der Azure-Region, in der Sie weiter oben in diesem Lab Ressourcen bereitgestellt haben|
    |Umgebungstyp|**Produktion**|
    |SAP-Produkt|**S/4HANA**|
    |Datenbank|**HANA**|
    |HANA-Skalierungsmethode|**Hochskalieren (empfohlen)**|
    |Bereitstellungstyp|**Verteilt mit Hochverfügbarkeit**|
    |Computeverfügbarkeit|**99,95 (Verfügbarkeitsgruppe)**|
    |Virtuelles Netzwerk|**CONTOSO-VNET**|
    |Anwendungssubnetz|**App (10.5.0.0/24)**|
    |Datenbanksubnetz|**Datenbank (10.5.2.0/24)**|
    |Anwendungsbetriebssystemimage|**Red Hat Enterprise Linux 8.4 für SAP-Anwendungen – x64 Gen2 neueste Version**|
    |Datenbankbetriebssystemimage|**Red Hat Enterprise Linux 8.4 für SAP-Anwendungen – x64 Gen2 neueste Version**|
    |SAP-Transportoption|**Erstellen eines neuen SAP-Transportverzeichnisses**|
    |Transportressourcengruppe|**ACSS-DEMO**|
    |Speicherkontoname|Kein Eintrag|
    |Authentication type|**Öffentliche SSH**|
    |Username|**contososapadmin**|
    |Quelle für öffentlichen SSH-Schlüssel|**Generieren eines neuen Schlüsselpaars**|
    |Schlüsselpaarname|**contosoc1skey**|
    |SQP-FQDN|**sap.contoso.com**|
    |Quelle für verwaltete Identität|**Vorhandene benutzerseitig zugewiesene verwaltete Identität verwenden**|
    |Name der verwalteten Identität|**Contoso-MSI**|

1. Geben Sie auf der Registerkarte **VMs** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |Empfehlung generieren basierend auf|**SAP Application Performance Standard (SAPS) – Wählen Sie diese Option aus, um einen SAPS-Wert für die Logikschicht sowie die Datenbankspeichergröße anzugeben, und klicken Sie auf „Empfehlungen generieren“.**|
    |SAPS für die Logikschicht|**10000**|
    |Arbeitsspeichergröße für Datenbank (GiB)|**1024**|

1. Wählen Sie **Empfehlung generieren** aus.
1. Überprüfen Sie die Größe und die Anzahl der VMs für ASCS, Anwendung und Datenbank. 

    >**Hinweis:** Passen Sie bei Bedarf die empfohlenen Größen an, indem Sie den Link **Alle Größen anzeigen** für jede Gruppe von VMs festlegen und eine alternative Größe auswählen. Standardmäßig führt der verteilte Bereitstellungstyp mit Hochverfügbarkeit sowie mit dem oben angegebenen SAPS-Wert für die Logikschicht und der oben angegebenen Datenbankspeichergröße zu den folgenden Empfehlungen für die VM-SKU:
    - 2 VMs vom Typ „Standard_E4ds_v4“ für die ASCS-VMs (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher)
    - 2 VMs vom Typ „Standard_E4ds_v4“ für die Anwendungs-VMs (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher)
    - 2 VMs vom Typ „Standard_M64ms“ für die Datenbank-VMs (jeweils 64 vCPUs und 1.750 GiB Arbeitsspeicher)

    >**Hinweis:** Um die vCPU- und Speicheranforderungen für die Datenbank-VMs zu minimieren, ändern Sie ggf. die VM-SKU in „Standard_M32ts“ (jeweils 32 vCPUs und 192 GiB Arbeitsspeicher).

    >**Hinweis:** Bei Bedarf können Sie eine Erhöhung des Kontingents anfordern, indem Sie den Link **Kontingent anfordern** für eine bestimmte SKU von VMs auswählen und eine Anforderung zur Kontingenterhöhung übermitteln. Die Verarbeitung einer Anforderung dauert in der Regel ein paar Minuten.

    >**Hinweis:** Azure Center for SAP solutions erzwingt die Verwendung der SAP-unterstützten VM-SKUs während der Bereitstellung.

1. Wählen Sie auf der Registerkarte **VMs** im Abschnitt **Datenträger** den Link **Konfiguration anzeigen und anpassen** aus.
1. Überprüfen Sie auf der Seite **Konfiguration des Datenbankdatenträgers** die empfohlene Konfiguration, ohne Änderungen vorzunehmen, und wählen Sie **Schließen** aus.
1. Navigieren Sie zurück zur Registerkarte **VMs**, und wählen Sie **Weiter: Architektur visualisieren** aus.
1. Überprüfen Sie auf der Registerkarte **Architektur visualisieren** das Diagramm, das die empfohlene Architektur veranschaulicht, und wählen Sie **Überprüfen + Erstellen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen + Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und aktivieren Sie das Kontrollkästchen, damit in der Bereitstellungsregion ausreichend Kontingent verfügbar ist, um zu vermeiden, dass der Fehler „Unzureichendes Kontingent“ auftritt. Wählen Sie anschließend **Erstellen** aus.
1. Wenn Sie dazu aufgefordert werden, wählen Sie im Fenster **Neues Schlüsselpaar generieren** die Option **Privaten Schlüssel herunterladen und Ressource erstellen** aus.

    >**Hinweis:** Der private Schlüssel, der zum Herstellen einer Verbindung mit den in der Bereitstellung enthaltenen Azure-VMs erforderlich ist, wird auf den Computer heruntergeladen, auf dem Sie dieses Lab ausführen.

    >**Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa 25 Minuten dauern.

>**Hinweis:** Fahren Sie nach der Bereitstellung entweder mit der Installation von SAP-Software fort, indem Sie Azure Center for SAP solutions verwenden, oder löschen Sie die Labressourcen, indem Sie die Anweisungen in der nächsten Aufgabe befolgen.

### Aufgabe 2: Löschen der in diesem Lab bereitgestellten Azure-Ressourcen

>**Wichtig:** Die Kosten der bereitgestellten Ressourcen sind erheblich. Stellen Sie daher sicher, dass Sie die Bereitstellung des Labs aufheben, wenn Sie es nicht weiter verwenden möchten. Durch das Löschen der virtuellen Instanz für SAP-Lösungen werden nicht die zugrunde liegenden Infrastrukturressourcen gelöscht. Zum Löschen der Ressourcen sollten Sie das in dieser Aufgabe beschriebene Verfahren verwenden, das auf Ressourcen in drei Ressourcengruppen ausgerichtet ist:

- **Contoso-SAP-C1S**
- **CONTOSO-VNET-RG**
- **ACSS-DEMO**

1. Wählen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, das Symbol **Cloud Shell** aus, und starten Sie eine PowerShell-Sitzung in Cloud Shell. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal in dem Azure-Abonnement starten, das Sie in diesem Lab verwenden, werden Sie aufgefordert, eine Azure-Dateifreigabe zum Speichern von Cloud Shell-Dateien zu erstellen. Akzeptieren Sie in diesem Fall die Standardwerte. Daraufhin wird ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt.

1. Führen Sie im Azure-Portal im Bereich **Cloud Shell** an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um alle in diesem Lab bereitgestellten Azure-VMs zu beenden und ihre Zuordnung aufzuheben:

    ```powershell
    $resourceGroupName = 'Contoso-SAP-C1S'
    $vms = Get-AzVM -ResourceGroupName $resourceGroupName
    foreach ($vm in $vms) {
       Stop-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name -Force 
    }
    ```

1. Führen Sie an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um alle Datenträger von allen in diesem Lab bereitgestellten Azure-VMs zu trennen:

    ```powershell
    foreach ($vm in $vms) {  
       $vmDisks = $vm.StorageProfile.DataDisks
       foreach ($vmDisk in $vmDisks) {
          Remove-AzVMDataDisk -VM $vm -Name $vmDisk.Name
       }
       Update-AzVM -ResourceGroupName $resourceGroupName -VM $vm -ErrorAction SilentlyContinue
    }
    ```

1. Führen Sie an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um die Löschoption für Netzwerkschnittstellen und Datenträger zu aktivieren, die an alle in diesem Lab bereitgestellten Azure-VMs angefügt sind:

    ```powershell
    foreach ($vm in $vms) {
       $vmConfig = Get-AzVM -ResourceGroupName $resourceGroupName -Name $vm.Name
       $vmConfig.StorageProfile.OsDisk.DeleteOption = 'Delete'
       $vmConfig.StorageProfile.DataDisks | ForEach-Object { $_.DeleteOption = 'Delete' }
       $vmConfig.NetworkProfile.NetworkInterfaces | ForEach-Object { $_.DeleteOption = 'Delete' }
       $vmConfig | Update-AzVM
    }   
    ```

1. Führen Sie an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um alle in diesem Lab bereitgestellten Azure-VMs zu löschen:

    ```powershell
    foreach ($vm in $vms) {
       Remove-AzVm -ResourceGroupName $resourceGroupName -Name $vm.Name -ForceDeletion $true -Force
    }
    ```

1. Führen Sie an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um die Ressourcengruppe **Contoso-SAP-C1S** und alle verbleibenden Ressourcen zu löschen:

    ```powershell
    Remove-AzResourceGroup -Name 'Contoso-SAP-C1S' -Force -AsJob
    ```

1. Führen Sie an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um die Ressourcengruppe **CONTOSO-VNET-RG** und alle verbleibenden Ressourcen zu löschen:

    ```powershell
    Remove-AzResourceGroup -Name 'CONTOSO-VNET-RG' -Force -AsJob
    ```

1. Führen Sie an der PowerShell-Eingabeaufforderung die folgenden Befehle aus, um die Ressourcengruppe **ACSS-DEMO** und alle verbleibenden Ressourcen zu löschen:

    ```powershell
    Remove-AzResourceGroup -Name 'ACSS-DEMO' -Force -AsJob
    ```

    >**Hinweis:** Die letzten drei Befehle werden (wie über Parameter „-AsJob“ festgelegt) asynchron ausgeführt. Dies bedeutet, dass Sie zwar direkt im Anschluss in derselben PowerShell-Sitzung den nächsten PowerShell-Befehl ausführen können, es jedoch einige Minuten dauert, bis die Ressourcengruppen und ihre Ressourcen tatsächlich entfernt werden.
