---
lab:
  title: 06c – Übersicht über die Bereitstellung von Azure Center für SAP-Lösungen
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# AZ 1006 Modul: Entwerfen und Implementieren einer Infrastruktur zur Unterstützung von SAP-Workloads in Azure
# AZ-1006 1-tägiges Kurs-Lab 6c: Übersicht über die Bereitstellung mit Azure Center für SAP-Lösungen'

Geschätzte Dauer: 60 Minuten

Alle Aufgaben in diesem AZ-1006 1-tägigen Kurs-Lab werden über das Azure-Portal durchgeführt

## Ziele

Nach Abschluss dieses Labs können Sie Folgendes:

- Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten, mithilfe des Azure Center für SAP-Lösungen

>**Wichtig:** Die in dieser Übung implementierten Voraussetzungen sollen *nicht* die bewährten Methoden für die Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center für SAP-Lösungen darstellen. Ihr Zweck besteht darin, Zeit, Kosten und Ressourcen zu minimieren, die erforderlich sind, um die Mechanismen der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions zu bewerten und Verwaltungs- und Wartungsaufgaben nach der Bereitstellung durchzuführen. Die Implementierung der Voraussetzungen umfasst die folgenden Aktivitäten:

- Erstellen einer durch Microsoft Entra zugewiesenen verwalteten Identität, die vom Azure Center für SAP-Lösungen für den Azure Storage-Zugriff während der Bereitstellung verwendet werden soll.
- Zuweisen der von Microsoft Entra zugewiesenen verwalteten Identität, die zum Ausführen des Bereitstellungszugriffs auf das Azure-Abonnement verwendet wird
- Erstellen des virtuellen Azure-Netzwerks, in dem alle virtuellen Azure-Computer gehostet werden, die in der Bereitstellung enthalten sind.

Diese Aktivitäten entsprechen den folgenden Aufgaben dieser Übung:

- Aufgabe 1: Erstellen einer benutzerseitig zugewiesenen verwalteten Microsoft Entra-Identität
- Aufgabe 2: Konfigurieren der RBAC-Rollenzuweisungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) in Azure für die benutzerseitig zugewiesene verwaltete Microsoft Entra ID-Identität
- Aufgabe 3: Erstellen des virtuellen Azure-Netzwerks

## Anweisungen

### Übung 1: Implementieren der Mindestvoraussetzungen für die Auswertung der Bereitstellung von SAP-Workloads in Azure mithilfe des Azure Center für SAP-Lösungen

Dauer: 15 Minuten

In dieser Übung implementieren Sie die reinen Mindestvoraussetzungen für die Auswertung der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center für SAP-Lösungen. Dazu gehören die folgenden Aktivitäten:

>**Wichtig:** Die in dieser Übung implementierten Voraussetzungen sollen *nicht* die bewährten Methoden für die Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions darstellen. Ihr Zweck besteht darin, Zeit, Kosten und Ressourcen zu minimieren, die erforderlich sind, um die Mechanismen der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions zu bewerten und Verwaltungs- und Wartungsaufgaben nach der Bereitstellung durchzuführen.

Die Implementierung der Voraussetzungen umfasst die folgenden Aktivitäten:

- Erstellen einer durch Microsoft Entra zugewiesenen verwalteten Identität, die vom Azure Center für SAP-Lösungen für den Azure-Zugriff während der Bereitstellung verwendet werden soll.
- Erstellen des virtuellen Azure-Netzwerks, in dem alle virtuellen Azure-Computer gehostet werden, die in der Bereitstellung enthalten sind.

Diese Aktivitäten entsprechen den folgenden Aufgaben dieser Übung:

- Aufgabe 1: Erstellen einer vom Microsoft Entra zugewiesenen verwalteten Identität
- Aufgabe 2: Erstellen des virtuellen Azure-Netzwerks

#### Aufgabe 1: Erstellen einer durch Microsoft Entra zugewiesenen verwalteten Identität

In dieser Aufgabe erstellen Sie eine benutzerseitig zugewiesene verwaltete Microsoft Entra-Identität, die von Azure Center for SAP solutions für den Azure Storage-Zugriff während der Bereitstellung verwendet werden soll.

1. Starten Sie auf dem Labcomputer einen Webbrowser, navigieren Sie zum Azure-Portal unter `https://portal.azure.com`, und authentifizieren Sie sich mithilfe eines Microsoft-Kontos oder eines Microsoft Entra ID-Kontos mit der Rolle „Besitzer“ bei dem Azure-Abonnement, das Sie in diesem Lab verwenden.
1. Suchen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Verwaltete Identitäten**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Verwaltete Identitäten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Benutzerseitig zugewiesene verwaltete Identität erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

    |Einstellung|Wert|
    |---|---|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Resource group|Name einer **neuen** Ressourcengruppe: **acss-infra-RG**|
    |Region|Name der Azure-Region, die Sie für die ACSS-Bereitstellung verwenden|
    |Name|**acss-infra-MI**|

1. Warten Sie auf der Registerkarte **Überprüfen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.
1. Warten Sie, bis der Bereitstellungsprozess abgeschlossen ist und wählen Sie **Zur Ressource wechseln**, um sich auf den nächsten Vorgang vorzubereiten. Die Bereitstellung sollte nur ein paar Sekunden dauern.

#### Aufgabe 2: Konfigurieren des Mitwirkenden des Azure-rollenbasierten Zugriffssteuerungsabonnements (RBAC)

1. Fahren Sie im Azure-Portal auf der Übersichtsseite „Verwaltete Identität“ für **acss-infra-MI** nach Abschluss der letzten Aufgabe fort.
1. Wählen Sie im Menü **acss-infra-MI** der Seite der verwalteten Identität **Azure-Rollenzuweisungen** aus.
1. Wählen Sie auf der Seite **Azure-Rollenzuweisungen** die Option **Rollenzuweisung hinzufügen** aus.

1. Geben Sie auf der Registerkarte **Rollenzuweisung hinzufügen** im Bereich **Rollenzuweisung hinzufügen** die folgenden Einstellungen an und **speichern** Sie:

    |Einstellung|Wert|
    |---|---|
    |Umfang|**Abonnement**|
    |Abonnement|Der Name des Azure-Abonnements, das in dieser Übung verwendet wird|
    |Rolle|**Mitwirkender**|

#### Aufgabe 3: Konfigurieren Sie rollenbasierte Zugriffssteuerungs-Rollenzuweisungen (Role-Based Access Control, RBAC) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird

##### Identität hinzufügen: „Azure Center für SAP-Lösungsdienstrolle“

1. Suchen Sie im Azure-Portal im Textfeld **Suchen** nach **Abonnements**.
1. Wählen Sie auf der Seite **Abonnements** den Eintrag aus, der das Azure-Abonnement darstellt, das Sie für dieses Lab verwenden werden. 
1. Wählen Sie auf der Seite, auf der die Eigenschaften des Azure-Abonnements angezeigt werden, die Option **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf der Seite **Zugriffssteuerung (IAM)** die Option **+ Hinzufügen** und dann im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Liste der **Auftragsfunktionsrollen** nach dem Eintrag **Azure Center for SAP solutions-Dienstrolle**, und wählen Sie die entsprechende Option aus. Wählen Sie anschließend **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Mitglieder** der Seite **Rollenzuweisung hinzufügen** für **Zugriff zuweisen auf** die **verwaltete Identität** aus und klicken Sie dann auf **+ Mitglieder auswählen**.
1. Geben Sie im Bereich **verwaltete Identitäten** auswählen die folgenden Einstellungen an und klicken Sie dann auf **Auswählen**:

    |Einstellung|Wert|
    |---|---|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Verwaltete Identität|**Benutzerseitig zugewiesene verwaltete Identität**|
    |Auswählen|**acss-infra-RG/subscriptions/...**|

1. Zurück auf der Registerkarte **Mitglieder**, wählen Sie **Überprüfen und Zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.

##### Identität hinzufügen: „Azure Center für SAP-Lösungsadministrator

1. Zurück auf dem Blatt **Zugriffssteuerung (IAM)** wählen Sie **+ Hinzufügen** aus und im Dropdownmenü klicken Sie auf **Rollenzuweisung hinzufügen**.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Liste der **Auftragsfunktionsrollen** nach dem Eintrag **Azure Center for SAP solutions-Administrator**, und wählen Sie die entsprechende Option aus. Wählen Sie anschließend **Weiter** aus.
1. Auf der Registerkarte **Mitglieder**
    - Wählen Sie unter **Zugriff zuweisen zu** die Option **Benutzer, Gruppe oder Dienstprinzipal** aus.
    - Klicken Sie auf **+ Mitglieder auswählen**.
1. Geben Sie im Bereich **Mitglieder auswählen** im Textfeld **Auswählen** den Namen des Microsoft Entra ID-Benutzerkontos ein, das Sie für den Zugriff auf das Azure-Abonnement verwendet haben, das Sie für dieses Lab verwenden, wählen Sie es in der Liste der Ergebnisse aus, die Ihrem Eintrag entsprechen, und klicken Sie dann auf **Auswählen**.
1. Navigieren Sie zurück zur Registerkarte **Mitglieder**, und wählen Sie **Überprüfen + zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.

##### Identität hinzufügen: „Operator für verwaltete Identität“

1. Zurück auf dem Blatt **Zugriffssteuerung (IAM)** wählen Sie **+ Hinzufügen** aus und im Dropdownmenü klicken Sie auf **Rollenzuweisung hinzufügen**.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Auflistung **Auftragsfunktionsrollen** nach dem Eintrag **Operator für verwaltete Identität**, und wählen Sie **Weiter** aus.
1. Auf der Registerkarte **Mitglieder**
    - Wählen Sie unter **Zugriff zuweisen zu** die Option **Benutzer, Gruppe oder Dienstprinzipal** aus.
    - Klicken Sie auf **+ Mitglieder auswählen**.
1. Geben Sie im Bereich **Mitglieder auswählen** im Textfeld **Auswählen** den Namen des Microsoft Entra ID-Benutzerkontos ein, das Sie für den Zugriff auf das Azure-Abonnement verwendet haben, das Sie für dieses Lab verwenden, wählen Sie es in der Liste der Ergebnisse aus, die Ihrem Eintrag entsprechen, und klicken Sie dann auf **Auswählen**.
1. Navigieren Sie zurück zur Registerkarte **Mitglieder**, und wählen Sie **Überprüfen + zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.

#### Aufgabe 4: Erstellen des virtuellen Netzwerks

In dieser Aufgabe erstellen Sie das virtuelle Azure-Netzwerk, in dem alle virtuellen Azure-Computer gehostet werden, die in der Bereitstellung enthalten sind. Darüber hinaus erstellen Sie im virtuellen Netzwerk die folgenden Subnetze:

- Bastionhost
- App – für das Hosten der SAP-Anwendung und der SAP Central Services-Instanzserver
- db – für das Hosten der SAP-Datenbankebene

1. Suchen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Virtuelle Netzwerke**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Virtuelle Netzwerke** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtuelles Netzwerk erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Resource group|**acss-infra-RG**|
    |Name des virtuellen Netzwerks|**acss-infra-VNET**|
    |Region|der Name der gleichen Azure-Region, die Sie in der vorherigen Aufgabe dieser Übung verwendet haben|

1. Aktivieren Sie auf der Registerkarte **Security** unter **Azure Bastion** das **Kontrollkästchen**, um **Azure Bastion zu aktivieren**.
1. Geben Sie die folgenden Einstellungen an, und wählen Sie **Weiter** aus.
    |Einstellung|Wert|
    |---|---|
    |Azure Bastion-Hostname|**acss-infra-vnet-bastion**|
    |Öffentliche Azure Bastion-IP-Adresse|**(Neu) acss-infra-bastion**|

1. Geben Sie auf der Registerkarte **IP-Adressen** die folgenden Einstellungen an und wählen Sie dann **hinzufügen** aus:

    |Einstellung|Wert|
    |---|---|
    |IP-Adressbereich|**/10.0.0.0/16 (65.536 Adressen)**|

1. Wählen Sie in der Liste der Subnetze das Papierkorbsymbol aus, um das **Standardsubnetz** zu **löschen**.

1. Wählen Sie **+ Subnetz hinzufügen** aus.
1. Geben Sie im Bereich **Subnetz** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus (übernehmen Sie für die anderen Einstellungen die Standardwerte):

    |Einstellung|Wert|
    |---|---|
    |Name|**acss-admin**|
    |Startadresse|**10.0.0.0**|
    |Größe|**/24 (256 Adressen)**|

1. Wählen Sie **+ Subnetz hinzufügen** aus.
1. Geben Sie im Bereich **Subnetz** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus (übernehmen Sie für die anderen Einstellungen die Standardwerte):

    |Einstellung|Wert|
    |---|---|
    |Name|**app**|
    |Startadresse|**10.0.2.0**|
    |Größe|**/24 (256 Adressen)**|

1. Wählen Sie **+ Subnetz hinzufügen** aus.
1. Geben Sie im Bereich **Subnetz** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus (übernehmen Sie für die anderen Einstellungen die Standardwerte):

    |Einstellung|Wert|
    |---|---|
    |Name|**db**|
    |Startadresse|**10.0.3.0**|
    |Größe|**/24 (256 Adressen)**|

1. Wählen Sie auf der Registerkarte **IP-Adressen** die Option **Überprüfen + Erstellen** aus:
1. Warten Sie auf der Registerkarte **Überprüfen und erstellen** bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie dann **Erstellen** aus.

    >**Hinweis:** Warten Sie bis zu ca. 3 Minuten, bis der Bereitstellungsprozess teilweise fortgesetzt wird, bevor Sie mit der nächsten Aufgabe fortfahren. Die gesamte Bereitstellung von Azure Bastion kann 25 Minuten dauern, also **warten wir nicht**.

### Übung 2: Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, unter Verwendung von Azure Center für SAP-Lösungen

Dauer: 20 Minuten

In dieser Übung führen Sie die Bereitstellung von Azure Center für SAP-Lösungen durch. Dazu gehören die folgenden Aktivitäten:

- Verwenden Sie Azure Center for SAP solutions, um die Infrastruktur bereitzustellen, die SAP-Workloads in einem Azure-Abonnement hosten kann.

Diese Aktivität entspricht der folgenden Aufgabe dieser Übung:

- Aufgabe 1: Erstellen einer Instanz von Virtual Instance for SAP (VIS) solutions

>**Hinweis:** Nach der erfolgreichen Bereitstellung können Sie mit der Installation von SAP-Software fortfahren, indem Sie Azure Center for SAP solutions verwenden. Die Installation von SAP-Software ist jedoch in diesem Lab nicht enthalten.

>**Hinweis:** Informationen zum Installieren von SAP-Software mithilfe von Azure Center for SAP solutions finden Sie in der Microsoft Learn-Dokumentation, in der beschrieben wird, wie Sie [SAP-Installationsmedien abrufen](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) und [SAP-Software installieren](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software).

#### Aufgabe 1: Erstellen einer Instanz von Virtual Instance for SAP (VIS) solutions

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Azure Center for SAP solutions**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie unter **Azure Center for SAP solutions \| Übersicht** die Option **Erstellen eines neuen SAP-Systems** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtual Instance for SAP solutions erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: VMs** aus.

    |Einstellung|Wert|
    |---|---|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Resource group|**acss-infra-RG**|
    |Name (SID)|**VI1**|
    |Region|der Name der Azure-Region, in der die ACSS-registrierte SAP-Bereitstellung gehostet wird oder eine andere Region in derselben Geografie|
    |Umgebungstyp|**Nicht-Produktion**|
    |SAP-Produkt|**S/4HANA**|
    |Datenbank|**HANA**|
    |HANA-Skalierungsmethode|**Hochskalieren (empfohlen)**|
    |Bereitstellungstyp|**Distributed**|
    |Virtuelles Netzwerk|**acss-infra-VNET**|
    |Anwendungssubnetz|**app**|
    |Datenbanksubnetz|**db**|
    |Imageoptionen für Anwendungsbetriebssysteme|**Verwenden eines Marketplace-Images**|
    |Anwendungsbetriebssystemimage|**Red Hat Enterprise Linux 8.6 für SAP-Anwendungen – x64 Gen2 neueste Version**|
    |Optionen für Datenbankbetriebssystemimages|**Verwenden eines Marketplace-Images**|
    |Datenbankbetriebssystemimage|**Red Hat Enterprise Linux 8.6 für SAP-Anwendungen – x64 Gen2 neueste Version**|
    |SAP-Transportoption|**Erstellen eines neuen SAP-Transportverzeichnisses**|
    |Transportressourcengruppe|**acss-infra-RG**|
    |Speicherkontoname|*leer*|
    |Authentication type|**Öffentliche SSH**|
    |Username|**contososapadmin**|
    |Quelle für öffentlichen SSH-Schlüssel|**Generieren eines neuen Schlüsselpaars**|
    |Schlüsselpaarname|**contosovi1key**|
    |SQP-FQDN|**sap.contoso.com**|
    |Quelle für verwaltete Identität|**Vorhandene benutzerseitig zugewiesene verwaltete Identität verwenden**|
    |Name der verwalteten Identität|**acss-infra-MI**|

1. Geben Sie auf der Registerkarte **VMs** die folgenden Einstellungen an:

    |Einstellung|Wert|
    |---|---|
    |Empfehlung generieren basierend auf|**SAP Application Performance Standard (SAPS) – Wählen Sie diese Option aus, um einen SAPS-Wert für die Logikschicht sowie die Datenbankspeichergröße anzugeben, und klicken Sie auf „Empfehlungen generieren“.**|
    |SAPS für die Logikschicht|**1000**|
    |Arbeitsspeichergröße für Datenbank (GiB)|**128**|

1. Wählen Sie **Empfehlung generieren** aus.
1. Überprüfen Sie die Größe und die Anzahl der VMs für ASCS, Anwendung und Datenbank.

    >**Hinweis:** Passen Sie bei Bedarf die empfohlenen Größen an, indem Sie den Link **Alle Größen anzeigen** für jede Gruppe von VMs festlegen und eine alternative Größe auswählen. Standardmäßig führt der verteilte Bereitstellungstyp mit hoher Verfügbarkeit sowie der oben angegebenen Größe des SAPS und der Datenbankspeichergröße der Anwendungsebene zu den folgenden Empfehlungen für die VM-SKU:
    - 1 x Standard_D4ds_v4 für die ASCS-VMs (jeweils 4 vCPUs und 16 GiB Arbeitsspeicher)
    - 1 x Standard_D4ds_v4 für die Anwendungs-VMs (jeweils 4 vCPUs und 16 GiB Arbeitsspeicher)
    - 1 x Standard_E16ds_v5 für die Datenbank-VMs (jeweils 16 vCPUs und 128 GiB des Arbeitsspeichers)

    >**Hinweis:** Bei Bedarf können Sie eine Erhöhung des Kontingents anfordern, indem Sie den Link **Anforderungskontingent** für eine bestimmte SKU virtueller Computer auswählen und eine Anforderung zur Erhöhung des Kontingents übermitteln. Die Verarbeitung einer Anforderung dauert in der Regel ein paar Minuten.

    >**Hinweis:** Azure Center for SAP solutions erzwingt die Verwendung der SAP-unterstützten VM-SKUs während der Bereitstellung.

1. Wählen Sie auf der Registerkarte **VMs** im Abschnitt **Datenträger** den Link **Konfiguration anzeigen und anpassen** aus.
1. Überprüfen Sie auf der Seite **Konfiguration des Datenbankdatenträgers** die empfohlene Konfiguration, ohne Änderungen vorzunehmen, und wählen Sie **Schließen** aus.
1. Navigieren Sie zurück zur Registerkarte **VMs**, und wählen Sie **Weiter: Architektur visualisieren** aus.
1. Überprüfen Sie auf der Registerkarte **Architektur visualisieren** das Diagramm, das die empfohlene Architektur veranschaulicht, und wählen Sie **Überprüfen + Erstellen** aus.
1. Warten Sie auf der Registerkarte **Überprüfen + Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und aktivieren Sie das Kontrollkästchen, damit in der Bereitstellungsregion ausreichend Kontingent verfügbar ist, um zu vermeiden, dass der Fehler „Unzureichendes Kontingent“ auftritt. Wählen Sie anschließend **Erstellen** aus.
1. Wenn Sie dazu aufgefordert werden, wählen Sie im Fenster **Neues Schlüsselpaar generieren** die Option **Privaten Schlüssel herunterladen und Ressource erstellen** aus.

    >**Hinweis:** Der private Schlüssel, der zum Herstellen einer Verbindung mit den in der Bereitstellung enthaltenen Azure-VMs erforderlich ist, wird auf den Computer heruntergeladen, von dem Sie diese Übung ausführen.

    >**Hinweis:** Warten Sie nur ca. 3 Minuten, bis der Bereitstellungsprozess teilweise fortgesetzt wird, bevor Sie mit der nächsten Aufgabe fortfahren. Die gesamte Bereitstellung kann 25 Minuten dauern, aber wir **warten nicht**.

    >**Hinweis:** Nach der Bereitstellung können Sie die SAP-Software mithilfe des Azure Center für SAP-Lösungen installieren. In dieser Übung werden Sie die Funktionen des Azure Center für SAP-Lösungen erkunden, ohne SAP-Software zu installieren.

### Übung 3: Erkunden eines VIS für SAP-Workloads in Azure mithilfe der Azure Center für SAP-Lösungen

Duration (Dauer): 25 Minuten

In dieser Übung zeigen Sie die Eigenschaften und Funktionen in einer Azure Center für SAP-Lösungen VIS an. Sie stellen auch eine Verbindung mit einem virtuellen Computer her, der von ACSS erstellt wurde, und erkunden die erstellten Verzeichnisse.

#### Aufgabe 1: Überprüfen der ACSS VIS-Seite

1. Nachdem die VIS-Bereitstellung abgeschlossen ist, zeigen Sie die Seite **VI1 Virtual Instance for SAP-Lösungen** an und erkunden Sie die verfügbaren Informationen aus dem Menü „Virtual Instance for SAP Solutions“, einschließlich:

    1. **Übersicht**
        - Die **Erste Schritte**-Registerkarte zeigt Optionen für „SAP-Software installieren“ und „Bereits installierte SAP-Software bestätigen“ an.
        - Die **Eigenschaften**-Registerkarte zeigt die bereitgestellten virtuellen Computer an.
        - Die Registerkarte **Überwachung** zeigt „CPU-Auslastung der zentralen Server-VM“, „Auslastung von Datenbankdatenträger-IOPS“ und „CPU-Auslastung der Datenbank-VM“ an.
    1. **Überwachen** > **Qualitätseinblicke** auf der Seite „Quality Insights“:
        - **Virtuelle Computer**: Erkunden Sie Compute List, Compute Extensions und Compute+OS Disk.
        - **Konfigurationsüberprüfungen**: Erkunden Sie die Unterelementoptionen: Beschleunigtes Netzwerk, Öffentliche IP, Sicherung und Lastenausgleich.
    1. **Kostenmanagement** > **Kostenanalyse**
        - Erweitern sie Elemente unter der Spalte **Ressource**.

#### Aufgabe 2: Herstellen einer Verbindung mit DB-VM und Überprüfen der ACSS-Konfiguration

1. Wählen Sie im Azure-Portal die virtuellen Computer aus und wählen Sie die Datenbank-VM aus, die in ACSS, **vi1dbvm** erstellt wurde.
1. Wählen Sie „Verbinden > Azure Bastion“ und dann die folgenden Einstellungen, und dann **Verbinden** aus:
    - Authentifizierungstyp **SSH Private Key aus lokaler Datei**
    - Username **contososapadmin** 
    - Lokale Datei ***privater Schlüssel, den Sie heruntergeladen haben***
    
1. Geben Sie in der Bash-Eingabeaufforderung Folgendes ein: `mount`, und suchen Sie die folgende Ausgabe für die Zuordnung:

    ```output
    /dev/sdb1 on /mnt type ext4 (rw,relatime,x-systemd.requires=cloud-init.service,_netdev)
    /dev/mapper/vg_sap-lv_usrsap on /usr/sap type xfs (rw,relatime,attr2,inode64,noquota)
    /dev/mapper/vg_hana_shared-lv_hana_shared on /hana/shared type xfs (rw,relatime,attr2,inode64,noquota)
    /dev/mapper/vg_hana_backup-lv_hana_backup on /hana/backup type xfs (rw,relatime,attr2,inode64,noquota)
    /dev/mapper/vg_hana_data-lv_hana_data on /hana/data type xfs (rw,relatime,attr2,inode64,sunit=512,swidth=2048,noquota)
    /dev/mapper/vg_hana_log-lv_hana_log on /hana/log type xfs (rw,relatime,attr2,inode64,sunit=128,swidth=384,noquota)
    ```

1. Geben Sie bei der Eingabeaufforderung Folgendes ein: `more /etc/fstab` und suchen Sie die Ausgabe ähnlich der Zuordnung für die SAP Media (`sapmedia`)-Freigabe:

    ```output
    10.100.1.8:/vi2nfs9fbec656c6a60a7/sapmedia on /usr/sap/install type nfs4     (rw,relatime,vers=4.1,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.100.1.6,local_
    lock=none,addr=10.100.1.8)
    ```

1. Geben Sie an der Eingabeaufforderung Folgendes ein: `cd /usr/sap/install`, um zum Verzeichnis zu navigieren, das `sapmedia` zugeordnet ist.

### Übung 4 (optional): Verwalten von SAP-Workloads in Azure mithilfe von Azure Center für SAP-Lösungen

Dauer: 20 Minuten

In dieser Übung überprüfen Sie die Verwaltung und Überwachung von SAP-Workloads nach der Bereitstellung mithilfe von Azure Center für SAP-Lösungen. Dazu gehören die folgenden Aktivitäten:

- Überprüfen der Voraussetzungen für die Sicherung von SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden
- Überprüfen der Voraussetzungen für die Notfallwiederherstellung von SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden
- Überprüfen von Überwachungsoptionen für SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden
- Löschen aller in diesem Lab bereitgestellten Azure-Ressourcen.

Diese Aktivitäten entsprechen den folgenden Aufgaben dieser Übung:

- Aufgabe 1: Überprüfen der Voraussetzungen für die Sicherung von SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden
- Aufgabe 2: Überprüfen der Voraussetzungen für die Notfallwiederherstellung von SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden
- Aufgabe 3: Überprüfen der Überwachungsoptionen für SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden
- Aufgabe 4: Löschen der in dieser Übung bereitgestellten Azure-Ressourcen

#### Aufgabe 1: Überprüfen der Voraussetzungen für die Sicherung von SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden

>**Hinweis:** Wenn Sie Azure Backup auf der VIS-Ressourcenebene im Azure Center für SAP-Lösungen konfigurieren, können Sie in einem Schritt die Sicherung für Azure-VMs, welche die Datenbank, Anwendungsserver und SAP Central Services-Instanz hosten, und für die HANA DB aktivieren. Für die HANA DB-Sicherung führt Azure Center for SAP solutions automatisch das Sicherungsvorregistrierungsskript aus.

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Azure Center for SAP solutions**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Azure Center for SAP solutions \| Übersicht** im vertikalen Navigationsmenü auf der linken Seite **Virtuelle Instanzen für SAP-Lösungen** und in der Liste der virtuellen Instanzen die Instanz aus, die Sie in der vorherigen Übung bereitgestellt haben.
1. Wählen Sie auf der Seite „Virtuelle Instanz“ im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Vorgänge** die Option **Sicherung (Vorschau)** aus.
1. Beachten Sie die Meldung, dass die Sicherung nicht eingerichtet werden kann, da die Installation bzw. Registrierung der SAP-Software für dieses SAP-System nicht abgeschlossen ist.

   >**Hinweis:** Dies entspricht dem erwarteten Verhalten. Sie können die Sicherung auf diese Weise erst einrichten, wenn die Installation der SAP-Software abgeschlossen ist. Das Abschließen des Setups beinhaltet jedoch auch zusätzliche Voraussetzungen, u. a. die Erstellung von Tresoren und Sicherungsrichtlinien. Wir sehen uns diese Voraussetzungen hier genauer an. 

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Backup Center**, und wählen Sie die entsprechende Option aus. 
1. Wählen Sie auf der Seite **Backup Center** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Verwalten** die Option **Tresore** aus.
1. Wählen Sie auf der Seite **Backup Center \| Tresore** die Option **+ Tresor** aus.
1. Überprüfen Sie auf der Seite **Start: Tresor erstellen** die verfügbaren Tresortypen, stellen Sie sicher, dass **Recovery Services-Tresor** (der die Datenquellentypen **Azure-VMs** und **SAP HANA in Azure VM** unterstützt) ausgewählt ist, und wählen Sie dann **Weiter** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Recovery Services-Tresor erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter: Redundanz** aus.

    |Einstellung|Wert|
    |---|---|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Resource group|Der Name einer **neuen** Ressourcengruppe: **acss-mgmt-RG**|
    |Name des Tresors|**acss-backup-RSV**|
    |Region|der Name der Azure-Region, in der die ACSS-registrierte SAP-Bereitstellung gehostet wird|

1. Geben Sie auf der Registerkarte **Redundanz** die folgenden Einstellungen an, und wählen Sie **Weiter: Tresoreigenschaften** aus.

    |Einstellung|Wert|
    |---|---|
    |Redundanz für Sicherungsspeicher|**Georedundant**|
    |Regionsübergreifende Wiederherstellung|**Aktivieren**|

1. Überprüfen Sie auf der Registerkarte **Tresoreigenschaften** die Einstellung **Unveränderlichkeit aktivieren**, ohne sie zu aktivieren, und wählen Sie **Weiter: Netzwerk** aus.
1. Übernehmen Sie auf der Registerkarte **Netzwerk** die Standardoption **öffentlichen Zugriff von allen Netzwerken zulassen** und wählen Sie dann **Überprüfen und erstellen** aus
1. Wählen Sie in dieser Übung **nicht** **Erstellen** aus, da wir nur überprüfen.
1. Warten Sie auf der Registerkarte **Überprüfen und Erstellen** bis der Überprüfungsprozess abgeschlossen ist, und kehren Sie dann zur Seite „Azure Center für SAP-Lösungen“ zurück (Einstellungen für die Sicherung gehen verloren).  

    >**Hinweis:** Das Feature [Sicherung (Vorschau)](https://learn.microsoft.com/azure/sap/center-sap-solutions/acss-backup-integration) von Azure Center für SAP-Lösungen-UI wird zu einer bevorzugten Methode zum Abschließen der Sicherungskonfiguration werden, wenn es als *Allgemein verfügbar* von *Preview* veröffentlicht wird.

    >**Hinweis:** Beim Konfigurieren der Sicherung auf VIS-Ebene im Azure Center für SAP-Lösungsschnittstellen können Sie den vorhandenen Tresor und seine Richtlinien nutzen.

    >**Hinweis:** Sobald die Sicherung auf VIS-Ebene konfiguriert ist, können Sie den Status von Sicherungsaufträgen von Azure VMs und HANA DB über die VIS-Schnittstelle im Azure-Portal überwachen.

#### Aufgabe 2: Überprüfen der Voraussetzungen für die Notfallwiederherstellung von SAP-Workloads, die vom Azure Center für SAP-Lösungen verwaltet werden

>**Hinweis:** Während der Azure Center für SAP-Lösungsdienst ein zonenredundanter Dienst ist, gibt es kein von Microsoft initiiertes Failover im Falle eines Regionsausfalls. Um dieses Szenario nachzubessern, sollten Sie die Notfallwiederherstellung für SAP-Systeme konfigurieren, die mithilfe von Azure Center for SAP solutions bereitgestellt werden. Befolgen Sie dazu die unter [Übersicht über die Notfallwiederherstellung und Infrastrukturrichtlinien für SAP-Workloads](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide) beschriebene Anleitung, die die Verwendung von Azure Site Recovery (ASR) umfasst. In dieser Aufgabe durchlaufen Sie den Prozess der Implementierung einer ASR-basierten Notfallwiederherstellungslösung, die auf dieser Anleitung basiert.

>**Hinweis:** ASR ist die empfohlene Lösung für Anwendungsserver und SAP Central Services-Instanzen. Bei Datenbankservern sollten Sie die native Replikationsfunktionalität in Betracht ziehen.

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Recovery Services-Tresore**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Recovery Services-Tresore** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Recovery Services-Tresor erstellen** die folgenden Einstellungen an (übernehmen Sie für die anderen Einstellungen die Standardwerte), und wählen Sie **Weiter: Redundanz** aus.

    |Einstellung|Wert|
    |---|---|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Resource group|Der Name einer **neuen** Ressourcengruppe: **acss-dr-RG**|
    |Name des Tresors|**acss-dr-RSV**|
    |Region|der Name der Azure-Region, die mit der ACSS-registrierten SAP-Bereitstellung gekoppelt ist|

    >**Hinweis:** Informationen zur Ermittlung der Region, die mit der Region gekoppelt ist, in der Ihre Produktionsworkloads gehostet werden, finden Sie in der MS Learn-Dokumentation mit der Beschreibung von [Azure-Regionspaare](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure#azure-paired-regions).

1. Geben Sie auf der Registerkarte **Redundanz** die folgenden Einstellungen an, und wählen Sie **Weiter: Tresoreigenschaften** aus.

    |Einstellung|Wert|
    |---|---|
    |Redundanz für Sicherungsspeicher|**Lokal redundant**|

1. Überprüfen Sie auf der Registerkarte **Tresoreigenschaften** die Einstellung **Unveränderlichkeit aktivieren**, ohne sie zu aktivieren, und wählen Sie **Weiter: Netzwerk** aus.
1. Übernehmen Sie auf der Registerkarte **Netzwerk** die Standardoption **Öffentlichen Zugriff aus allen Netzwerken zulassen**, und wählen Sie dann **Überprüfen + Erstellen** aus
1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

    >**Hinweis:** Warten Sie nicht, bis der Bereitstellungsprozess abgeschlossen ist, sondern fahren Sie stattdessen mit dem nächsten Schritt fort. Die Bereitstellung kann ungefähr 2 Minuten dauern.

    >**Hinweis:** Jetzt richten Sie die Notfallwiederherstellungsumgebung in der gekoppelten Region ein, in der Sie den Recovery Services-Tresor erstellt haben. Diese Umgebung enthält ein virtuelles Netzwerk mit gehosteten Replikaten der Azure-VMs, die derzeit in der primären Region gehostet werden, in der Sie Virtual Instance for SAP bereitgestellt haben. 

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Recovery Services-Tresore**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Recovery Services-Tresore** die Option **acss-dr-RSV** aus.
1. Wählen Sie auf der Seite **acss-dr-RSV** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Erste Schritte** die Option **Site Recovery** aus.
1. Wählen Sie auf der Seite **acss-dr-RSV \| Site Recovery** im Abschnitt **Azure-VMs** die Option **1. Aktivieren der Replikation**. 
1. Legen Sie auf der Registerkarte **Quelle** der Seite **Replikation aktivieren** die folgenden Einstellungen an, und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Region|Name der Azure-Region, in der Virtual Instance for SAP (VIS) gehostet wird|
    |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
    |Resource group|**acss-vi-RG**|
    |Bereitstellungsmodell für virtuelle Computer|**Ressourcen-Manager**|
    |Notfallwiederherstellung zwischen Verfügbarkeitszonen|**Nein**|

    >**Hinweis:** Die **Notfallwiederherstellung zwischen Verfügbarkeitszonen** ist möglicherweise nicht konfigurierbar, je nachdem, ob die Quellregion Verfügbarkeitszonen unterstützt.

1. Wählen Sie auf der Registerkarte **Virtuelle Computer** die ersten vier virtuellen Computer in der Liste (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0** und **vi1ascsvm0**) und dann **Weiter** aus.

    >**Hinweis:** Wie bereits erwähnt, wird die ASR-basierte Replikation auf die Anwendungsserver und SAP Central Services-Instanzen angewendet. Datenbankserver werden mithilfe der nativen Datenbankreplikationsfunktion synchronisiert.

1. Führen Sie auf der Registerkarte **Replikationseinstellungen** die folgenden Aktionen aus:

    1. Wählen Sie bei Bedarf in der Dropdownliste **Zielspeicherort** die Azure-Region aus, in der Sie den Recovery Services-Tresor **acss-dr-RSV** erstellt haben.
    1. Stellen Sie sicher, dass der Name des in diesem Lab verwendeten Azure-Abonnements in der Dropdownliste **Zielabonnement** angezeigt wird.
    1. Wählen Sie in der Dropdownliste **Zielressourcengruppe** die Option **acss-dr-RG** aus.
    1. Wählen Sie unter der Dropdownliste **Virtuelles Failovernetzwerk** die Option **Neu erstellen** aus.
    1. Geben Sie im Bereich **Virtuelles Netzwerk erstellen** im Textfeld **Name** den Namen **CONTOSO-VNET-asr** ein
    1. Ersetzen Sie im Abschnitt **Adressraum** im Textfeld **Adressbereich** den Standardeintrag durch **10.10.0.0/16**.
    1. Geben Sie im Abschnitt **Subnetze** im Textfeld **Subnetzname** den Namen **App** ein, und geben Sie im Textfeld **Adressbereich** den Bereich **10.10.0.0/24** ein.
    1. Geben Sie unter dem neu hinzugefügten Subnetzeintrag im Abschnitt **Subnetze** im Textfeld **Subnetzname** den Namen **db** ein, und geben Sie im Textfeld **Adressbereich** den Bereich **10.10.2.0/24** ein.
    1. Wählen Sie im Bereich **Virtuelles Netzwerk erstellen** die Option **OK** aus.
    1. Navigieren Sie zurück zur Seite **Replikation aktivieren **, und vergewissern Sie sich, dass der Eintrag **(neue) App (10.10.0.0/24)** in der Dropdownliste **Failover-Subnetz** angezeigt wird.
    1. Wählen Sie im Abschnitt **Speicher** den Link **Speicherkonfiguration anzeigen/bearbeiten** aus.
    1. Überprüfen Sie auf der Seite **Zieleinstellungen anpassen** die resultierende Konfiguration, nehmen Sie aber keine Änderungen vor, und wählen Sie **Abbrechen** aus.
    1. Wählen Sie im Abschnitt **Verfügbarkeitsoptionen** den Link **Verfügbarkeitsoptionen anzeigen/bearbeiten** aus.
    1. Auf der Seite **Verfügbarkeitsoptionen** haben Sie die Möglichkeit, Näherungsplatzierungsgruppen für Zielressourcen zu implementieren. Nehmen Sie aber keine Änderungen vor, und wählen Sie **Abbrechen** aus.

    >**Hinweis:** Sie haben auch die Möglichkeit, die Kapazitätsreservierung zu konfigurieren.

    >**Hinweis:** Sie haben auch die Möglichkeit, Replikationsrichtlinieneinstellungen zu konfigurieren.

    >**Wichtig:** Beachten Sie, dass sich die IP-Adressräume zwischen dem virtuellen Netzwerk in den primären und sekundären Regionen unterscheiden. Dies ist beabsichtigt, da es die Verbindung zwischen den beiden virtuellen Netzwerken ermöglicht. Das ist erforderlich, um die Replikation zwischen Datenbankservern zu konfigurieren, die in den beiden Regionen gehostet werden. Diese Verbindung kann mithilfe des virtuellen Netzwerk-Peerings hergestellt werden.

1. Wählen Sie auf der Registerkarte **Replikationseinstellungen** der Seite **Replikation aktivieren** **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Verwalten** auf der Seite **Replikation aktivieren** die Option **Weiter** aus.
1. Überprüfen Sie auf der Registerkarte **Überprüfen** auf der Seite **Replikation aktivieren** die Einstellungen. Für diese Übung wird die Replikation **nicht aktiviert**.

    >**Hinweis:** Die Erstreplikation dauert in der Regel erhebliche Zeit bis zum Abschluss. Unter Berücksichtigung der begrenzten Zeit, die diesem Lab zugeteilt ist, wenden Sie sich an den Kursleiter bezüglich zusätzlicher Schritte, die als Teil dieser Aufgabe ausgeführt werden sollen. Wenn keine spezifischen Anweisungen vorhanden sind, fahren Sie direkt mit der nächsten Aufgabe fort.

    >**Hinweis:** Sie können zu diesem Zeitpunkt sowohl Azure Bastion als auch Azure Firewall bereitstellen, aber Sie sollten die Bereitstellung stattdessen im Rahmen der Failoverprozedur für die Notfallwiederherstellung automatisieren. Dadurch werden die Gebühren für die Aufrechterhaltung der Notfallwiederherstellungsumgebung minimiert. Das Gleiche gilt für andere Komponenten dieser Umgebung, welche die Konfiguration der primären Instanz von Virtual Instance for SAP widerspiegeln, z. B. Azure-Premium-Dateifreigaben und benutzerdefiniertes Routing.

#### Aufgabe 3: Überprüfen der Überwachungsoptionen für SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden

>**Hinweis:** Wie bei der Sicherung können Sie die Überwachungsfunktionen von Azure Center for SAP solutions nicht in vollem Umfang nutzen. Dies erfordert die Installation von SAP-Software oder das Registrieren einer vorhandenen Instanz für Azure Monitor for SAP solutions Stattdessen nutzen Sie in dieser Aufgabe die in Virtual Instance for SAP solutions verfügbare Schnittstelle, um diese Funktionen zu identifizieren und zu überprüfen.

1. Suchen Sie auf dem Labcomputer im Microsoft Edge-Fenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Virtual Instances for SAP solutions**, und wählen Sie die entsprechende Option aus. 
1. Überprüfen Sie auf der Seite **Virtual Instances for SAP solutions** die zusammengefassten Statusinformationen für die **VI1**-Instanz, einschließlich der visuellen Indikatoren für die allgemeine Integrität und den Status.
1. Wählen Sie auf der Seite **Virtual Instances for SAP solutions** die Option **VI1** aus.
1. Wählen Sie auf der Seite **VI1** im vertikalen Navigationsmenü auf der linken Seite **Übersicht** und dann im Bereich auf der rechten Seite **Überwachung** aus.
1. Überprüfen Sie die im Überwachungsbereich angezeigte Überwachungstelemetrie.

    >**Hinweis:** Der Überwachungsbereich enthält vCPU-Auslastungsdiagramme und Metriken für Anwendungsserver, Datenbankserver und SAP Central Services-Instanzen. Er enthält auch Datenbankserver für die IOPS-Statistik von Datenbankdatenträgern. 

1. Wählen Sie auf der Seite **VI1** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Überwachung** die Option **Qualitätserkenntnisse** aus.
1. Überprüfen Sie auf der Seite **VI1 \| Qualitätserkenntnisse \| Arbeitsmappe 1** die Registerkarte **Advisor-Empfehlung**, die Empfehlungen für die Optimierung von Virtual Instance for SAP solutions (VIS), Central-Serverinstanzen, App-Dienstinstanzen und Datenbanken enthält.

    >**Hinweis:** Für diese Empfehlungen ist die Installation von SAP-Software erforderlich.

1. Wählen Sie auf der Seite **VI1 \| Qualitätserkenntnisse \| Arbeitsmappe 1** die Registerkarte **VM** aus, und überprüfen Sie den Inhalt der Registerkarten **Azure Compute**, **Computeliste**, **Computeerweiterungen**, **Compute- und Betriebssystemdatenträger** und **Compute + Datenträger**.

    >**Hinweis:** Jede dieser Registerkarten sollte tatsächliche Daten enthalten, die auf den Azure-VMs gesammelt werden, die Teil von Virtual Instance for SAP solutions sind.

1. Wählen Sie auf der Seite **VI1 \| Qualitätserkenntnisse \| Arbeitsmappe 1** die Registerkarte **Konfigurationsüberprüfungen** aus, und überprüfen Sie den Inhalt der Registerkarten **Beschleunigter Netzwerkbetrieb**, **Öffentliche IP-Adresse**, **Sicherung** und **Load Balancer**. Dieser Inhalt bietet einen schnellen Überblick über die Leistungs- und Sicherheitseinstellungen von Compute- und Netzwerkkomponenten von Virtual Instance for SAP solutions. Die Registerkarte **Load Balancer** enthält Informationen von **Load Balancer-Monitor**, die wichtige Lastenausgleichsmetriken anzeigen.
1. Wählen Sie auf der Seite **VI1** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Überwachung** die Option **Azure Monitor für SAP-Lösungen** aus.
1. Beachten Sie auf der Seite **VI1 \| Azure Monitor für SAP-Lösungen** die Meldung, dass AMS nicht eingerichtet werden kann, da die Installation bzw. Registrierung der SAP-Software für VIS nicht abgeschlossen ist.

    >**Hinweis:** Nachdem Sie SAP-Software installiert haben, können Sie sie in eine neue oder vorhandene Ressource für Azure Monitor für SAP-Lösungen integrieren. Azure Monitor für SAP-Lösungen basiert auf den Azure Monitor-Funktionen von Log Analytics und Arbeitsmappen, um eine umfassende Überwachung von SAP-Workloads bereitzustellen, die auf Azure-VMs gehostet werden, einschließlich Unterstützung für benutzerdefinierte Visualisierungen, Abfragen und Warnungen.
