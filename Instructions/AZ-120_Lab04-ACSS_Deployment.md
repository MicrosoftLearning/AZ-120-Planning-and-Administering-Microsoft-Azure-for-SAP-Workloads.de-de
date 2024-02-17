---
lab:
  title: Automatisieren der Bereitstellung mithilfe von Azure Center for SAP solutions
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# AZ 120 Modul: Entwerfen und Implementieren einer Infrastruktur zur Unterstützung von SAP-Workloads in Azure
# Lab: Automatisieren der Bereitstellung mithilfe von Azure Center für SAP-Lösungen

Geschätzte Dauer: 100 Minuten

Alle Aufgaben in dieses Labs werden über das Azure-Portal ausgeführt

## Ziele

In diesem Lab lernen Sie Folgendes:

- Implementieren von Voraussetzungen für die Bereitstellung von SAP-Workloads in Azure mithilfe des Azure Center für SAP-Lösungen
- Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, mithilfe von Azure Center für SAP-Lösungen

## Übung 1: Implementieren von Voraussetzungen für die Bereitstellung von SAP-Workloads in Azure mithilfe des Azure Center für SAP-Lösungen

Duration (Dauer): 60 Minuten

In dieser Übung implementieren Sie die Voraussetzungen für die Bereitstellung von SAP-Workloads in Azure mithilfe des Azure Center for SAP-Lösungen. Dazu gehören die folgenden Aufgaben:
- Erfüllen der vCPU-Anforderungen im Azure-Zielabonnement
- Konfigurieren der rollenbasierten Zugriffssteuerungs-Rollenzuweisungen (Role-Based Access Control, RBAC) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird
- Erstellen eines Speicherkontos, das dem für die Bereitstellung verwendeten Azure Center for SAP solutions zugeordnet ist
- Erstellen einer benutzerseitig zugewiesenen verwalteten Identität, die vom Azure Center for SAP solutions für die Authentifizierung und Autorisierung ihrer automatisierten Bereitstellung verwendet werden soll
- Erstellen einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die in Subnetzen des virtuellen Netzwerks verwendet werden soll, das die Bereitstellung hostet
- Erstellen von Routingtabellen, die in Subnetzen des virtuellen Netzwerks verwendet werden sollen, das die Bereitstellung hostet
- Erstellen und Konfigurieren des virtuellen Netzwerks, das die Bereitstellung hosten soll
- Bereitstellen der Azure Firewall im virtuellen Netzwerk, das die Bereitstellung hosten wird
- Bereitstellen von Azure Bastion im virtuellen Netzwerk, das die Bereitstellung hosten wird

Die Übung umfasst die folgenden Aufgaben:

- Aufgabe 1: Erfüllen der vCPU-Anforderungen im Azure-Zielabonnement
- Aufgabe 2: Konfigurieren der rollenbasierten Zugriffssteuerungs-Rollenzuweisungen (Role-Based Access Control, RBAC) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird
- Aufgabe 3: Erstellen eines Speicherkontos, das dem für die Bereitstellung verwendeten Azure Center for SAP-Lösungen zugeordnet ist
- Aufgabe 4: Erstellen und konfigurieren einer benutzerseitig zugewiesenen verwalteten Identität, die vom Azure Center for SAP-Lösungen für die Authentifizierung und Autorisierung ihrer automatisierten Bereitstellung verwendet werden soll
- Aufgabe 5: Erstellen einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die in Subnetzen des virtuellen Netzwerks verwendet werden soll, das die Bereitstellung hostet
- Aufgabe 6: Erstellen von Routingtabellen, die in Subnetzen des virtuellen Netzwerks verwendet werden sollen, das die Bereitstellung hostet
- Aufgabe 7: Erstellen und konfigurieren des virtuellen Netzwerks, das die Bereitstellung hostet
- Aufgabe 8: Bereitstellen der Azure Firewall in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird
- Aufgabe 9: Bereitstellen von Azure Bastion in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird

### Aufgabe 1: Erfüllen der vCPU-Anforderungen im Azure-Zielabonnement

>**Hinweis:** Um diese Übung abzuschließen (wie beschrieben), benötigen Sie ein Microsoft Azure-Abonnement mit den vCPU-Kontingenten, welche die Bereitstellung der folgenden virtuellen Computer berücksichtigen:

- 2 x Standard_E4ds_v4 (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher) oder 2 X Standard_D4ds_v4 (4 vCPUs und je 16 GiB des Arbeitsspeichers) VMs für die ASCS-Ebene
- 2 x Standard_E4ds_v4 (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher) oder 2 X Standard_D4ds_v4 (4 vCPUs und je 16 GiB Arbeitsspeicher) VMs für die Anwendungsebene 
- 2 x Standard_M64ms (64 vCPUs und je 1750 GiB Arbeitsspeicher) VMs für die Datenbankebene

>**Hinweis:** Um die vCPU- und Speicheranforderungen für die virtuellen Computer der Datenbank zu minimieren, können Sie die VM-SKU in Standard_M32ts ändern (32 vCPUs und je 192 GiB des Arbeitsspeichers).

1. Starten Sie auf dem Lab-Computer einen Webbrowser und navigieren Sie zum Azure-Portal unter `https://portal.azure.com`.
1. Wählen Sie im Azure-Portal das **Cloud Shell**-Symbol aus und starten Sie eine PowerShell-Sitzung in Cloud Shell. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal im Azure-Abonnement starten, das Sie in dieser Übung verwenden, werden Sie aufgefordert, eine Azure-Dateifreigabe zum Speichern von Cloud Shell-Dateien zu erstellen. Akzeptieren Sie in diesem Falls die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Azure-Portal im **Cloud Shell**-Bereich bei der PowerShell-Eingabeaufforderung Folgendes aus: Dabei legt `<Azure_region>` die Azure-Region fest, in der Sie Ressourcen in dieser Übung bereitstellen möchten (z. B. `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **Hinweis:** Um die Namen von Azure-Regionen zu identifizieren, führen Sie `(Get-AzLocation).Location` in der **Cloud Shell** an der Bash-Eingabeaufforderung durch

1. Überprüfen Sie die Ausgabe, um die aktuelle vCPU-Verwendung und den vCPU-Grenzwert zu identifizieren. Stellen Sie sicher, dass der Unterschied zwischen ihnen ausreicht, um vCPUs von Azure-VMs aufzunehmen, die Sie in diesem Lab bereitstellen werden. Berücksichtigen Sie sowohl VM-familienspezifische als auch regionale vCPU-Nummern. 
1. Wenn die Anzahl der vCPUs nicht ausreicht, schließen Sie den Cloud Shell-Bereich im Azure-Portal im Textfeld **Suchen**, und wählen Sie **Kontingente** aus.
1. Wählen Sie auf der Seite **Kontingente** **Compute** aus.
1. Verwenden Sie auf der Seite **Kontingente \| Compute** den Filter **Region**, um die Azure-Region auszuwählen, in der Sie Ressourcen in dieser Übung bereitstellen möchten.
1. Suchen Sie in der Spalte **Kontingentname** den VM-SKU-Namen, der eine Kontingenterhöhung erfordert, und wählen Sie ihn aus. 
1. Überprüfen Sie in derselben Zeile den Eintrag in der Spalte **Anpassbar**. Der nächste Schritt hängt davon ab, ob die Spalte **Ja** oder **Nein** Eintrag enthält.

   - Wenn der Eintrag auf **Ja** festgelegt ist, wählen Sie das Symbol **Anforderungsanpassung** auf der **Neuen Kontingentanforderung**im Textfeld **Neuer Grenzwert** das neue Kontingentlimit ein, und wählen Sie dann **Absenden** aus.
   - Wenn der Eintrag auf **Nein** festgelegt ist, wählen Sie die Option **Zugriff anfordern oder Empfehlungen abrufen** und den Bereich **Kontingentempfehlungen** aus, klicken Sie auf **Support kontaktieren** und dann auf **Weiter**. 
1. Geben Sie in der Registerkarte **Problembeschreibung** der Seite **Neue Supportanfrage** die folgenden Einstellungen an und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Worauf bezieht sich Ihr Problem?|**Azure-Dienste**|
    |Problemtyp|**Grenzwerte für Dienste und Abonnements (Kontingente)**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Kontingenttyp|**Grenzwert für Compute/VM (Cores/vCPUs) erhöht**|

1. Wählen Sie auf der Registerkarte **Weitere Details** die Option **Details eingeben** aus.
1. Wählen Sie auf der Registerkarte **Kontingentdetails** in der Dropdownliste **Bereitstellungsmodell** **Ressourcen-Manager** aus, wählen Sie dann in der Dropdownliste **Speicherorte** die Ziel-Azure-Region in der Dropdownliste **Kontingente** aus, klicken Sie danach auf die Azure VM-Serie, für die Sie die Kontingentbeschränkungen erhöhen müssen, und geben Sie im Textfeld **Neuen Grenzwert** den neuen Kontingentgrenzwert ein und wählen Sie dann **Speichern und fortfahren** aus.
1. Zurück auf der Registerkarte **Zusätzliche Details** wählen Sie auf der Registerkarte **Erweiterte Diagnoseinformationen** **Ja (empfohlen)** aus.
1. Wählen Sie im Abschnitt **Supportmethode** entweder **E-Mail** oder **Phone** als bevorzugte Kontaktmethode aus, und wählen Sie dann **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.

    > **Hinweis:** Warten Sie, bis die Anforderung zum Erhöhen der Kontingentbeschränkungen erfolgreich abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren.

### Aufgabe 2: Konfigurieren der rollenbasierten Zugriffssteuerungs-Rollenzuweisungen (Role-Based Access Control, RBAC) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird

1. Starten Sie auf dem Lab-Computer Microsoft Edge und navigieren Sie zum Azure-Portal unter `https://portal.azure.com`.
1. Wenn Sie zur Authentifizierung aufgefordert werden, melden Sie sich mit den Microsoft Entra-ID-Anmeldeinformationen mit der Rolle „Besitzer“ im Azure-Abonnement an, das Sie für diese Übung verwenden. 
1. Suchen Sie im Azure-Portal im Textfeld **Suchen** nach **Abonnements**.
1. Wählen Sie auf der Seite **Abonnements** den Eintrag aus, der das Azure-Abonnement darstellt, das Sie für diese Übung verwenden werden. 
1. Wählen Sie auf der Seite, auf der die Eigenschaften des Azure-Abonnements angezeigt werden, die **Zugriffssteuerung (IAM) **aus.
1. Wählen Sie auf der Seite **Access Control (IAM)** **+ Hinzufügen** aus und wählen Sie dann im Dropdownmenü **Rollenzuweisung hinzufügen** aus.
1. Suchen und wählen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Liste der Rollen **Auftragsfunktionsrollen** aus, suchen Sie nach dem **Azure Center für SAP-Lösungsadministrator**-Eintrag, und wählen Sie **Weiter** aus.
1. Klicken Sie auf der Registerkarte **Mitglieder** der Seite **Rollenzuweisung hinzufügen** auf **+ Mitglieder auswählen**. 
1. Geben Sie im Bereich **Mitglieder auswählen** im Textfeld **Auswählen** den Namen des Microsoft Entra ID-Benutzerkontos ein, das Sie für den Zugriff auf das Azure-Abonnement verwendet haben, das Sie für diese Übung verwenden, wählen Sie es in der Liste der Ergebnisse aus, die Ihrem Eintrag entsprechen, und klicken Sie dann auf **Auswählen**.
1. Zurück auf der Registerkarte **Mitglieder**, wählen Sie **Überprüfen und Zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.
1. Wiederholen Sie die vorherigen sechs Schritte, um die Rolle des **Managed Identity Operator** dem Benutzerkonto zuzuweisen, das Sie für diese Übung verwenden.

### Aufgabe 3: Erstellen eines Speicherkontos, das dem für die Bereitstellung verwendeten Azure Center for SAP-Lösungen zugeordnet ist

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach dem **Speicherkonto** und wählen Sie es aus.
1. Wählen Sie auf der Seite **Speicherkonten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Seite **Speicherkonto erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an und wählen Sie **Weiter: Erweitert >**.

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|der Name einer **neuen** Ressourcengruppe **ACSS-DEMO**|
    |Speicherkontoname|Ein beliebiger global eindeutiger Name, der zwischen 3 und 24 Zeichen lang ist und aus Buchstaben und Ziffern besteht.|
    |Region|**USA, Osten**|
    |Leistung|**Standard**|
    |Redundanz|**Georedundanter Speicher (GRS)**|
    |Bei regionaler Verfügbarkeit Lesezugriff auf die Daten bereitstellen|Deaktiviert|

1. Überprüfen Sie auf der Registerkarte **Erweitert** die verfügbaren Optionen, übernehmen Sie die Standardwerte und wählen Sie **Weiter: Netzwerk >** aus.
1. Überprüfen Sie auf der Registerkarte **Netzwerk** die verfügbaren Optionen, stellen Sie sicher, dass die Option **Aktivieren des öffentlichen Zugriffs aus allen Netzwerken** aktiviert ist und wählen Sie **Überprüfen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie nicht, bis die Bereitstellung des Azure Storage-Kontos abgeschlossen ist. Fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefähr 2 Minuten dauern.

### Aufgabe 4: Erstellen und konfigurieren einer benutzerseitig zugewiesenen verwalteten Identität, die vom Azure Center for SAP-Lösungen für die Authentifizierung und Autorisierung ihrer automatisierten Bereitstellung verwendet werden soll

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach den **Verwalteten Identitäten** und wählen Sie sie aus.
1. Wählen Sie auf der Seite **Verwaltete Identitäten** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Vom Benutzer zugewiesene verwaltete Identität erstellen** die folgenden Einstellungen an und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**ACSS-DEMO**|
    |Region|**USA, Osten**|
    |Name|**Contoso-MSI**|

1. Warten Sie auf der Registerkarte **Überprüfen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie, bis die Bereitstellung der vom Benutzer zugewiesenen verwalteten Identität abgeschlossen ist. Dies sollte nur ein paar Sekunden dauern.

1. Navigieren Sie im Azure-Portal zur Seite **Verwaltete Identitäten** und wählen Sie den Eintrag **Contoso-MSI** aus.
1. Wählen Sie auf der Seite **Contoso-MSI** **Azure-Rollenzuweisungen** aus.
1. Wählen Sie auf der Seite **Azure-Rollenzuweisungen** **+ Rollenzuweisung hinzufügen (Vorschau)** aus.
1. Geben Sie auf der Seite **+ Rollenzuweisung hinzufügen (Vorschau)** die folgenden Einstellungen an und klicken Sie auf **Speichern**:

    |Einstellung|Wert|
    |---|---|
    |Umfang|**Abonnement**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Rolle|**Azure Center für SAP-Lösungsdienstrolle**|

2. Zurück auf der Seite **Azure-Rollenzuweisungen** wählen Sie **+ Rollenzuweisung hinzufügen (Vorschau)** aus.
3. Geben Sie auf der Seite **+ Rollenzuweisung hinzufügen (Vorschau)** die folgenden Einstellungen an und klicken Sie auf **Speichern**:

    |Einstellung|Wert|
    |---|---|
    |Umfang|**Storage**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Ressource|Der Name des Azure Storage-Kontos, das Sie in der vorherigen Aufgabe erstellt haben|
    |Rolle|**Lese- und Datenzugriff**|

### Aufgabe 5: Erstellen einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die in Subnetzen des virtuellen Netzwerks verwendet werden soll, das die Bereitstellung hostet

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Netzwerksicherheitsgruppen** und wählen Sie sie aus.
1. Wählen Sie auf der Seite **Netzwerksicherheitsgruppen** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Netzwerksicherheitsgruppe erstellen** die folgenden Einstellungen an und wählen Sie dann **Überprüfen und erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer **neuen** Ressourcengruppe **CONTOSO-VNET-RG**|
    |Name|**ACSS-DEMO-NSG**|
    |Region|**USA, Osten**|

1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen**aus.

    >**Hinweis:** Standardmäßig ermöglichen die integrierten Regeln von Netzwerksicherheitsgruppen den gesamten ausgehenden Datenverkehr, den gesamten Datenverkehr innerhalb desselben virtuellen Netzwerks sowie den gesamten Datenverkehr zwischen virtuellen Peer-Netzwerken. Dies reicht aus, um das Lab erfolgreich abzuschließen. Je nach Ihren Sicherheitsanforderungen sollten Sie möglicherweise einen Teil dieses Datenverkehrs blockieren. Wenn das der Fall ist, lesen Sie die Anleitungen in der Microsoft Learn-Dokumentation [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network).

### Aufgabe 6: Erstellen von Routingtabellen, die in Subnetzen des virtuellen Netzwerks verwendet werden sollen, das die Bereitstellung hostet

1. Suchen Sie auf dem Laborcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Routentabellen**.
1. Wählen Sie auf der Seite **Routentabellen** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Routingtabelle erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|**USA, Osten**|
    |Name|**ACSS-ROUTE**|
    |Gatewayrouten verteilen|**Nein**|

1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen**aus.

### Aufgabe 7: Erstellen und konfigurieren des virtuellen Netzwerks, das die Bereitstellung hostet

1. Suchen Sie auf dem Laborcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Virtuellen Netzwerken**. 
1. Wählen Sie auf der Seite **Virtuelle Netzwerke** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen eines virtuellen Netzwerks** die folgenden Einstellungen an und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name des virtuellen Netzwerks|**CONTOSO-VNET**|
    |Region|**(USA) USA, Osten**|

1. Übernehmen Sie auf der Registerkarte **Sicherheit** die Standardeinstellungen und wählen Sie **Weiter** aus.

    >**Hinweis:** Sie können zu diesem Zeitpunkt sowohl Azure Bastion als auch Azure Firewall bereitstellen, sie werden jedoch separat bereitgestellt, nachdem das virtuelle Netzwerk erstellt wurde.

1. Geben Sie auf der Registerkarte **IP-Adressen** die folgenden Einstellungen an und wählen Sie dann **Überprüfen und erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |IP-Adressbereich|**/10.5.0.0/16 (65.536 Adressen)**|

    >**Hinweis:** Löschen Sie alle zuvor erstellten Subnetzeinträge. Sie fügen Subnetze hinzu, nachdem das virtuelle Netzwerk erstellt wurde.

1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen**aus.
1. Navigieren Sie zurück zur Seite **Virtuelle Netzwerke** und wählen Sie den Eintrag **CONTOSO-VNET** aus. 
1. Wählen Sie auf der Seite **CONTOSO-VNET** in der vertikalen Menüleiste auf der linken Seite **Subnetze** aus.
1. Wählen Sie auf der Seite **CONTOSO-VNET \| Subnetze** **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz** hinzufügen die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**app**|
    |Subnetzadressbereich|**10.5.0.0/24**|
    |Netzwerksicherheitsgruppe|**ACSS-DEMO-NSG**|
    |Routingtabelle|**ACSS-ROUTE**|

1. Zurück auf der Seite **CONTOSO-VNET \| Subnetze** wählen Sie **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz** hinzufügen die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**AzureBastionSubnet**|
    |Subnetzadressbereich|**10.5.1.0/26**|

1. Zurück auf der Seite **CONTOSO-VNET \| Subnetze** wählen Sie **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz** hinzufügen die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**db**|
    |Subnetzadressbereich|**10.5.2.0/24**|
    |Netzwerksicherheitsgruppe|**ACSS-DEMO-NSG**|
    |Routingtabelle|**ACSS-ROUTE**|

1. Zurück auf der Seite **CONTOSO-VNET \| Subnetze** wählen Sie **+ Subnetz** aus. 
1. Geben Sie im Bereich **Subnetz** hinzufügen die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

    |Einstellung|Wert|
    |---|---|
    |Name|**AzureFirewallSubnet**|
    |Subnetzadressbereich|**10.5.3.0/24**|

### Aufgabe 8: Bereitstellen der Azure Firewall in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird

>**Hinweis:** Bevor Sie eine Azure Firewall-Instanz bereitstellen, erstellen Sie zuerst eine Firewallrichtlinie und eine öffentliche IP-Adresse, die von der Instanz verwendet werden soll.

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Firewallrichtlinien**.
1. Wählen Sie auf der Seite **Firewallrichtlinien** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen einer Azure-Firewallrichtlinie** die folgenden Einstellungen an und wählen Sie **Weiter: DNS-Einstellungen >**:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**FirewallPolicy_contoso-firewall**|
    |Region|**USA, Osten**|
    |Richtlinientarif|**Standard**|
    |Übergeordnete Richtlinie|**Keine**|

1. Übernehmen Sie auf der Registerkarte **DNS-Einstellungen** die Standardoption **Deaktiviert** und wählen Sie **Weiter: TLS-Inspektion >**.
1. Wählen Sie auf der Registerkarte **TLS-Inspektion** **Weiter: Regeln >**.
1. Wählen Sie auf der Registerkarte **Regeln** **+ Regelauflistung hinzufügen** aus.
1. Geben Sie im Bereich **Hinzufügen einer Regelsammlung** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |Name|**AllowOutbound**|
    |Regelsammlungstyp|**Netzwerk**|
    |Priority|**101**|
    |Regelsammlungsaktion|**Zulassen**|
    |Regelsammlungsgruppe|**DefaultNetworkRuleCollectionGroup**|

1. Fügen Sie im Bereich **Eine Regelsammlung hinzufügen** im Abschnitt **Regeln** eine Regel mit den folgenden Einstellungen hinzu:

    |Einstellung|Wert|
    |---|---|
    |Name|**RHEL**|
    |Quellentyp|**IP-Adresse**|
    |Quelle|*|
    |Protokoll|**Beliebige**|
    |Zielports|*|
    |Zieltyp|**IP-Adresse**|
    |Destination|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|

    >**Hinweis:** Informationen zur Identifizierung der für RHEL zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

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

    >**Hinweis:** Informationen zur Identifizierung der für SUSE zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

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
1. Wählen Sie zurück auf der Registerkarte **Regeln** **Weiter: IDPS >**.
1. Wählen Sie auf der Registerkarte **IDPS** **Weiter: Threat Intelligence >**.

    >**Hinweis:** Für die IDPS-Funktionalität ist die Premium-SKU erforderlich.

1. Überprüfen Sie auf der Registerkarte **Threat Intelligence** die verfügbaren Einstellungen, ohne Änderungen vorzunehmen, und wählen Sie dann **Überprüfen + erstellen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen**aus.

    >**Hinweis:** Warten Sie, bis die Bereitstellung der Firewallrichtlinie abgeschlossen ist. Die Bereitstellung sollte ungefähr 1 Minuten dauern.

1. Suchen Sie auf dem Laborcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Öffentlichen IP-Adressen**.
1. Wählen Sie auf der Seite **Öffentliche IP-Adressen** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen einer öffentlichen IP-Adresse** die folgenden Einstellungen an und wählen Sie dann **Überprüfen und erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Region|**(USA) USA, Osten**|
    |Name|**contoso-firewal-pip**|
    |IP-Version|**IPv4**|
    |SKU|**Standard**|
    |Verfügbarkeitszone|**Keine Zone**|
    |Tarif|**Regional**|
    |Routingpräferenz|**Microsoft Network**|
    |Leerlaufzeitüberschreitung (Minuten)|**4**|
    |DNS-Namensbezeichnung|nicht festgelegt|

1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen**aus.

    >**Hinweis:** Warten Sie, bis die Bereitstellung der öffentlichen IP-Adresse abgeschlossen ist. Die Bereitstellung sollte einige Sekunden dauern.

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Firewalls**.
1. Wählen Sie auf der Seite **Firewalls** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen einer Firewall** die folgenden Einstellungen an und wählen Sie dann **Überprüfen und erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**contoso-firewall**|
    |Region|**USA, Osten**|
    |Verfügbarkeitszone|**Keine**|
    |Firewall-SKU|**Standard**|
    |Firewallverwaltung|**Firewallrichtlinie zum Verwalten dieser Firewall verwenden**|
    |Firewallrichtlinie|**FirewallPolicy_contoso-firewall**|
    |Virtuelles Netzwerk auswählen|**Vorhandenes verwenden**|
    |Virtuelles Netzwerk|**CONTOSO-VNET**|
    |Öffentliche IP-Adresse|**contoso-firewall-pip**|
    |Tunnelerzwingung|**Disabled**|

    >**Hinweis:** Warten Sie, bis die Bereitstellung der Azure-Firewall abgeschlossen ist. Die Bereitstellung kann ungefähr 3 Minuten dauern.

1. Navigieren Sie im Azure-Portal zurück zur Seite **Firewalls**.
1. Wählen Sie auf der Seite **Firewalls** den Eintrag **contoso-firewall** aus.
1. Notieren Sie sich auf der Seite **contoso-firewall** den Eintrag **Private IP** auf **10.5.3.4**, der die private IP-Adresse der Azure Firewall-Instanz darstellt.

    >**Hinweis:** Damit der Netzwerkdatenverkehr über die Azure-Firewall weitergeleitet werden kann, müssen Sie benutzerdefinierte Routen zu den Routentabellen hinzufügen, die der App und db-Subnetzen des virtuellen Netzwerks zugeordnet sind, welche die SAP-Bereitstellung hosten.

1. Suchen Sie im Azure-Portal im Textfeld **Suchen** nach **Routingtabellen** und wählen Sie sie aus.
1. Wählen Sie auf der Seite **Routingabellen** den Eintrag **ACSS-ROUTE** aus.
1. Wählen Sie auf der Seite **ACSS-ROUTE** **Routen** aus.
1. Wählen Sie auf der Seite **ACSS-ROUTE \| Routes** **+ hinzufügen** aus.
1. Geben Sie im Bereich **Route hinzufügen** die folgenden Einstellungen an und wählen Sie dann **Hinzufügen** aus:

    |Einstellung|Wert|
    |---|---|
    |Routenname|**Firewall**|
    |Zieltyp|**IP-Adressen**|
    |Ziel-IP-Adressen/CIDR-Bereiche|**0.0.0.0/0**|
    |Typ des nächsten Hops|**Virtuelles Gerät**|
    |Adresse des nächsten Hops|**10.5.3.4**|

### Aufgabe 9: Bereitstellen von Azure Bastion in dem virtuellen Netzwerk, in dem die Bereitstellung gehostet wird

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Bastions** und wählen Sie sie aus. 
1. Wählen Sie auf der Seite **Bastions** **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Bastions** die folgenden Einstellungen an und wählen Sie **Weiter: Tags >**:

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|**CONTOSO-VNET-RG**|
    |Name|**ACSS-BASTION**|
    |Region|**USA, Osten**|
    |Tarif|**Grundlegend**|
    |Anzahl von Instanzen|**2**|
    |Virtuelles Netzwerk|**CONTOSO-VNET**|
    |Subnetz|**AzureBastionSubnet**|
    |Öffentliche IP-Adresse|**Neu erstellen**|
    |Name der öffentlichen IP-Adresse|**ACSS-BASTION-PIP**|

1. Wählen Sie auf der Registerkarte **Tags** **Weiter: Erweitert >**.
1. Überprüfen Sie auf der Registerkarte **Erweitert** die verfügbaren Einstellungen, ohne Änderungen vorzunehmen, und wählen Sie dann **Weiter: Überprüfen und erstellen >**.
1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen**aus.

    >**Hinweis:** Warten Sie nicht, bis die Bereitstellung des Bastion-Hosts abgeschlossen ist. Fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefähr 15 Minuten dauern.

## Übung 2: Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, mithilfe von Azure Center für SAP-Lösungen

Duration (Dauer): 40 Minuten

In dieser Übung verwenden Sie Azure Center für SAP-Lösungen, um die Infrastruktur bereitzustellen, die SAP-Workloads in dem Azure-Abonnement hosten wird, das Sie in der vorherigen Übung verwendet haben.
Die Übung besteht aus der folgenden Aufgabe:

- Aufgabe 1: Virtual Instance für SAP-Lösungen erstellen

### Aufgabe 1: Virtual Instance für SAP-Lösungen erstellen

1. Suchen Sie auf dem Lab-Computer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Azure Center für SAP-Lösungen**. 
1. Wählen Sie auf der Seite **Azure Center für SAP-Lösungen \| Übersicht** die Option **Neues SAP-System erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen einer Virtual Instance für SAP-Lösungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Virtuelle Computer**

    |Einstellung|Wert|
    |---|---|
    |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Resource group|Der Name einer **neuen** Ressourcengruppe **Contoso-SAP-C1S**|
    |Name (SID)|**C1S**|
    |Region|**(USA) USA, Osten**|
    |Umgebungstyp|**Produktion**|
    |SAP-Produkt|**S/4HANA**|
    |Datenbank|**HANA**|
    |HANA-Skalierungsmethode|**Hochskalieren (empfohlen)**|
    |Bereitstellungstyp|**Verteilt mit Hochverfügbarkeit (HA)**|
    |Berechnen der Verfügbarkeit|**99.95 (Verfügbarkeitssatz)**|
    |Virtuelles Netzwerk|**CONTOSO-VNET**|
    |Anwendungssubnetz|**App (10.5.0.0/24)**|
    |Datenbanksubnetz|**db (10.5.2.0/24)**|
    |Image für Anwendungsbetriebssystem|**Red Hat Enterprise Linux 8.2 für SAP-Anwendungen - x64 Gen2 neueste Version**|
    |Image für Datenbankbetriebssystem|**Red Hat Enterprise Linux 8.2 für SAP-Anwendungen - x64 Gen2 neueste Version**|
    |SAP-Transportoption|**Erstellen eines neuen SAP-Transportverzeichnisses**|
    |Transportressourcengruppe|**ACSS-DEMO**|
    |Speicherkontoname|Kein Eintrag|
    |Authentication type|**Öffentlichen SSH**|
    |Username|**contososapadmin**|
    |Quelle für öffentlichen SSH-Schlüssel|**Generieren eines neuen Schlüsselpaars**|
    |Schlüsselpaarname|**contosoc1skey**|
    |SQP-FQDN|**sap.contoso.com**|
    |Quelle für verwaltete Identität|**Vorhandene vom Benutzer zugewiesene verwaltete Identität verwenden**|
    |Name der verwalteten Identität|**Contoso-MSI**|

1. Geben Sie auf der Registerkarte **virtuelle Computer** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |Empfehlung generieren basierend auf|**SAP Application Performance Standard (SAPS) – Wählen Sie diese Option aus, um SAPS für Anwendungsebene und Datenbankspeichergröße bereitzustellen und klicken Sie auf „Empfehlungen generieren“**|
    |SAPS für die Anwendungsebene|**10000**|
    |Arbeitsspeichergröße für Datenbank (GiB)|**1024**|

1. Wählen Sie **Empfehlung generieren** aus.
1. Überprüfen Sie die Größe und die Anzahl der virtuellen Computer für die virtuellen Computer ASCS, Anwendung und Datenbank. 

    >**Hinweis:** Passen Sie bei Bedarf die empfohlenen Größen an, indem Sie den Link **Alle Größen anzeigen** für jede Gruppe virtueller Computer festlegen und eine alternative Größe auswählen. Standardmäßig führt der verteilte Bereitstellungstyp mit hoher Verfügbarkeit sowie der oben angegebenen Größe des SAPS und der Datenbankspeichergröße der Anwendungsebene zu den folgenden Empfehlungen für die VM-SKU:
    - 2 x Standard_E4ds_v4 für die ASCS-VMs (4 vCPUs und je 32 GiB Arbeitsspeicher)
    - 2 x Standard_E4ds_v4 VMs für die VMs der Anwendung (4 vCPUs und je 32 GiB Arbeitsspeicher)
    - 2 x Standard_M64ms für die Datenbank-VMs (64 vCPUs und je 1750 GiB des Arbeitsspeichers)

    >**Hinweis:** Um die vCPU- und Speicheranforderungen für die virtuellen Computer der Datenbank zu minimieren, könnten Sie die VM-SKU in Standard_M32ts ändern (32 vCPUs und je 192 GiB des Arbeitsspeichers).

    >**Hinweis:** Bei Bedarf können Sie eine Erhöhung des Kontingents anfordern, indem Sie den Link **Anforderungskontingent** für eine bestimmte SKU virtueller Computer auswählen und eine Anforderung zur Erhöhung des Kontingents übermitteln. Die Verarbeitung einer Anforderung dauert in der Regel ein paar Minuten.

    >**Hinweis:** Azure Center für SAP-Lösungen erzwingt die Verwendung der SAP-unterstützten VM-SKUs während der Bereitstellung.

1. Wählen Sie auf der Registerkarte **Virtuelle Computer** im Abschnitt **Datenträger** den Link **Anzeigen und Anpassen der Konfiguration** aus.
1. Überprüfen Sie auf der Konfigurationsseite des **Datenbankdatenträgers** die empfohlene Konfiguration, ohne Änderungen vorzunehmen, und wählen Sie **Schließen** aus.
1. Zurück auf der Registerkarte **Virtuelle Computer**, wählen Sie **Weiter: Architektur visualisieren**.
1. Überprüfen Sie auf der Registerkarte **Visualisieren der Architektur** das Diagramm, das die empfohlene Architektur veranschaulicht, und wählen Sie **Überprüfen + erstellen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen + Erstellen** bis der Überprüfungsprozess abgeschlossen ist, aktivieren Sie das Kontrollkästchen, damit in der Bereitstellungsregion ausreichend Kontingent verfügbar ist, um zu vermeiden, dass der Fehler „Unzureichendes Kontingent“ auftritt, und wählen Sie **Erstellen** aus.
1. Wenn Sie dazu aufgefordert werden, wählen Sie im Fenster **Neues Schlüsselpaar generieren** **Privaten Schlüssel herunterladen und Ressourcen erstellen** aus.

    >**Hinweis:** Der private Schlüssel, der zum Herstellen einer Verbindung mit den in der Bereitstellung enthaltenen Azure-VMs erforderlich ist, wird auf den Computer heruntergeladen, von dem Sie diese Übung ausführen.

    >**Hinweis**: Warten Sie, bis die Bereitstellung abgeschlossen ist. Dies kann etwa 25 Minuten dauern.

>**Hinweis:** Fahren Sie nach der Bereitstellung mit dem nächsten Schritt fort, bei dem SAP-Software mithilfe von Azure Center für SAP-Lösungen installiert wird.

>**Wichtig:** Die Kosten der bereitgestellten Ressourcen sind erheblich. Stellen Sie daher sicher, dass Sie die Bereitstellung des Labs aufheben, wenn Sie es nicht weiter verwenden möchten. Beachten Sie, dass das Löschen der virtuellen Instanz für SAP-Lösungen die zugrunde liegenden Infrastrukturressourcen nicht löscht. Um die Ressourcen zu löschen, sollten Sie die folgenden Ressourcengruppen, die im Verlauf dieser Übung erstellt wurden, (in dieser Reihenfolge) löschen:

- **Contoso-SAP-C1S**
- **CONTOSO-VNET-RG**
- **ACSS-DEMO**