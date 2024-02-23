---
lab:
  title: 06a – Übersicht über die Voraussetzungen für die Bereitstellung von Azure Center for SAP solutions (ACSS)
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# AZ 1006 Modul: Entwerfen und Implementieren einer Infrastruktur zur Unterstützung von SAP-Workloads in Azure
# AZ-1006 1-tägiges Kurslabor: Übersicht über die Voraussetzungen für die Bereitstellung von SAP-Workloads mit Azure Center für SAP-Lösungen (ACSS)

Geschätzte Dauer: 100 Minuten

Alle Aufgaben in diesem 1-tägigen Kurslab AZ-1006 werden über das Azure-Portal durchgeführt.

## Ziele

Nach Abschluss dieses Labs können Sie Folgendes:

- Implementieren von Voraussetzungen zum Bereitstellen von SAP-Workloads in Azure unter Verwendung von Azure Center for SAP solutions

## Anweisungen

### Übung 1: Implementieren von Voraussetzungen zum Bereitstellen von SAP-Workloads in Azure unter Verwendung von Azure Center for SAP solutions

Duration (Dauer): 60 Minuten

In dieser Übung überprüfen und implementieren Sie die Voraussetzungen für die Bereitstellung von SAP-Workloads in Azure mithilfe des Azure Center für SAP-Lösungen. Der Vorgang umfasst folgende Aktionen:

- Erstellen einer vom Microsoft Entra zugewiesenen verwalteten Identität, die vom Azure Center für SAP-Lösungen für Den Azure Storage-Zugriff während der Bereitstellung verwendet werden soll.
- Erstellen des virtuellen Azure-Netzwerks, in dem alle virtuellen Azure-Computer gehostet werden, die in der Bereitstellung enthalten sind.
- Erstellen einer Azure Bastion-Ressource zum Sichern der Konnektivität mit Azure-VMs aus dem Internet.
- Erstellen eines Azure Storage General Purpose v2-Kontos, das dem Azure Center for SAP solutions zugeordnet ist, die für die Bereitstellung verwendet werden
- Gewähren der vom Microsoft Entra zugewiesenen verwalteten Identität, die zum Ausführen des Bereitstellungszugriffs auf das Azure-Abonnement und das Azure Storage General Purpose v2-Konto verwendet wird
- Erstellen eines Azure Premium-Dateifreigabekontos, das zum Implementieren des SAP-Transportverzeichnisses verwendet wird
- Erstellen und Konfigurieren einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die verwendet wird, um ausgehenden Zugriff von Subnetzen des virtuellen Netzwerks einzuschränken, das die Bereitstellung hostet.
- Erstellen eines virtuellen Azure-Computers (VM), der für die SAP-Softwareinstallation als Teil einer Bereitstellung von Azure Center for SAP solutions verwendet werden soll.
- Herstellen einer Verbindung mit dem virtuellen Azure-Computer mithilfe von Azure Bastion und Konfigurieren für die SAP-Softwareinstallation.
- Löschen aller in dieser Übung bereitgestellten Azure-Ressourcen.

Diese Aktivitäten entsprechen den folgenden Aufgaben dieser Übung:

- Aufgabe 1: Erstellen einer vom Microsoft Entra zugewiesenen verwalteten Identität
- Aufgabe 2: Virtuelles Azure-Netzwerk erstellen
- Aufgabe 3: Erstellen einer Azure Bastion-Ressource
- Aufgabe 4: Erstellen eines Azure Storage General Purpose v2-Kontos
- Aufgabe 5: Konfigurieren der Autorisierung der vom Microsoft Entra zugewiesenen verwalteten Identität
- Aufgabe 6: Erstellen eines Azure Premium-Dateifreigabekontos
- Aufgabe 7: Erstellen und Konfigurieren einer Netzwerksicherheitsgruppe
- Aufgabe 8 Erstellen eines virtuellen Azure-Computers
- Aufgabe 9 Konfigurieren des virtuellen Azure-Netzwerks
- Aufgabe 10 Entfernen Sie Azure-Ressourcen,

#### Aufgabe 1: Erstellen einer vom Microsoft Entra zugewiesenen verwalteten Identität

In dieser Aufgabe erstellen Sie eine benutzerseitig zugewiesene verwaltete Microsoft Entra-Identität, die von Azure Center for SAP solutions für den Azure Storage-Zugriff während der Bereitstellung verwendet werden soll.

1. Starten Sie auf dem Labcomputer einen Webbrowser, navigieren Sie zum Azure-Portal unter `https://portal.azure.com`, und authentifizieren Sie sich mithilfe eines Microsoft-Kontos oder eines Microsoft Entra ID-Kontos mit der Rolle „Besitzer“ bei dem Azure-Abonnement, das Sie in diesem Lab verwenden.
1. Suchen Sie im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Verwaltete Identitäten**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Verwaltete Identitäten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Benutzerseitig zugewiesene verwaltete Identität erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
`   |Resource group| Auswählen oder Erstellen neuer **acss-infra-RG**|
   |Region|der Name der Azure-Region, die Sie für die ACSS-Bereitstellung verwenden|
   |Name|**acss-infra-MI**|

1. Warten Sie auf der Registerkarte **Überprüfen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie nicht, bis der Bereitstellungsprozess abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung sollte nur ein paar Sekunden dauern.

   >**Hinweis:** In einer der anstehenden Aufgaben autorisieren Sie den Zugriff auf die verwaltete Identität auf das Speicherkonto, das die SAP-Installationsmedien hosten, um die Installation von SAP-Software über das Azure Center for SAP solutions aufzunehmen.

#### Aufgabe 2: Erstellen des virtuellen Netzwerks

In dieser Aufgabe erstellen Sie das virtuelle Azure-Netzwerk, in dem alle virtuellen Azure-Computer gehostet werden, die in der Bereitstellung enthalten sind. Darüber hinaus erstellen Sie im virtuellen Netzwerk die folgenden Subnetze:

- AzureFirewallSubnet – für die Bereitstellung von Azure Firewall vorgesehen
- AzureBastionSubnet – für die Bereitstellung von Azure Bastion vorgesehen
- dmz – für die Bereitstellung der Azure-VM vorgesehen, die für die Bereitstellung von SAP-Software verwendet wird
- app – für das Hosten der SAP-Anwendung und der SAP Central Services-Instanzserver
- db – für das Hosten der SAP-Datenbankebene

1. Suchen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Virtuelle Netzwerke**, und wählen Sie die entsprechende Option aus. 
1. Wählen Sie auf der Seite **Virtuelle Netzwerke** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Virtuelles Netzwerk erstellen** die folgenden Einstellungen an, und wählen Sie **Weiter** aus:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**acss-infra-RG**|
   |Name des virtuellen Netzwerks|**acss-infra-VNET**|
   |Region|der Name der gleichen Azure-Region, die Sie in der vorherigen Aufgabe dieser Übung verwendet haben|

1. Übernehmen Sie auf der Registerkarte **Sicherheit** die Standardeinstellungen, und wählen Sie **Weiter** aus.

   >**Hinweis:** Sie können zu diesem Zeitpunkt sowohl Azure Bastion als auch Azure Firewall bereitstellen, sie werden jedoch separat bereitgestellt, nachdem das virtuelle Netzwerk erstellt wurde.

1. Geben Sie auf der **Registerkarte "IP-Adressen** " die folgenden Subnetzeinstellungen an, und wählen Sie dann **"Überprüfen + Erstellen"** aus:

   |Einstellung|Wert|
   |---|---|
   |IP-Adressbereich|**/10.0.0.0/16 (65.536 Adressen)**|

1. Wählen Sie in der Liste der Subnetze das Papierkorbsymbol aus, um das **Standardsubnetz** zu löschen.
1. Wählen Sie **+ Subnetz hinzufügen** aus.
1. Geben Sie im Bereich **Subnetz** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus (übernehmen Sie für die anderen Einstellungen die Standardwerte):

   |Einstellung|Wert|
   |---|---|
   |Subnetzzweck|**Azure Firewall**|
   |Startadresse|**10.0.0.0**|

   >**Hinweis:** Dadurch wird dem Subnetz automatisch der Name **AzureFirewallSubnet** zugewiesen und seine Größe auf **/26 (64 Adressen)** festgelegt.

1. Wählen Sie **+ Subnetz hinzufügen** aus.
1. Geben Sie im Bereich **Subnetz** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus (übernehmen Sie für die anderen Einstellungen die Standardwerte):

   |Einstellung|Wert|
   |---|---|
   |Name|**dmz**|
   |Startadresse|**10.0.0.128**|
   |Größe|**/26 (64 Adressen)**|

   >**Hinweis:** Dieses Subnetz wird verwendet, um die Azure-VM zu hosten, die Sie zum Installieren der SAP-Software über das Azure Center for SAP solutions verwenden.

1. Wählen Sie **+ Subnetz hinzufügen** aus.
1. Geben Sie im Bereich **Subnetz** die folgenden Einstellungen an, und wählen Sie dann **Hinzufügen** aus (übernehmen Sie für die anderen Einstellungen die Standardwerte):

   |Einstellung|Wert|
   |---|---|
   |Subnetzzweck|**Azure Bastion**|
   |Startadresse|**10.0.1.0**|
   |Größe|**/24 (256 Adressen)**|

   >**Hinweis:** Dadurch wird dem Subnetz automatisch der Name **AzureBastionSubnet**zugewiesen.

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
1. Warten Sie auf der Registerkarte **Überprüfen + Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie nicht, bis der Bereitstellungsprozess abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung sollte nur ein paar Sekunden dauern.

#### Aufgabe 3: Erstellen einer Azure Bastion-Ressource

In dieser Aufgabe erstellen Sie eine Azure Bastion-Ressource, um die Verbindung mit Azure-VMs aus dem Internet zu sichern.

1. Suchen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen** " nach **Bastionen** und wählen Sie sie aus. 
1. Wählen Sie auf der **Seite Bastions** + **Erstellen aus**.
1. Geben Sie auf der **Registerkarte "Grundlagen** " der **Seite "Bastions** " die folgenden Einstellungen an, und wählen Sie "Weiter" aus **: Erweitert>**:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**acss-infra-RG**|
   |Name|**acss-infra-BASTION**|
   |Region|der Name der gleichen Azure-Region, die Sie zuvor in dieser Übung verwendet haben|
   |Tarif|**Grundlegend**|
   |Anzahl von Instanzen|**2**|
   |Virtuelles Netzwerk|**acss-infra-VNET**|
   |Subnetz|**AzureBastionSubnet (10.0.1.0/24)**|
   |Öffentliche IP-Adresse|**Neu erstellen**|
   |Name der öffentlichen IP-Adresse|**acss-bastion-PIP**|

1. Überprüfen Sie auf der **Registerkarte "Erweitert** " die verfügbaren Optionen, ohne Änderungen vorzunehmen, und wählen Sie **dann "Überprüfen+ erstellen**" aus.
1. Warten Sie auf der **Registerkarte "Überprüfen+ Erstellen** ", bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie dann **"Erstellen"** aus.

   >**Hinweis:** Warten Sie nicht, bis der Bereitstellungsvorgang abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung kann ungefähr 5 Minuten dauern.

#### Aufgabe 4: Erstellen eines Azure Storage General Purpose v2-Kontos

In dieser Aufgabe erstellen Sie ein Azure Storage General Purpose v2-Konto, das dem Azure Center for SAP solutions zugeordnet ist, die für die Bereitstellung verwendet werden. Dieses Speicherkonto wird verwendet, um die SAP-Installationsmedien für die Installation von SAP-Software über das Azure Center for SAP solutions zu hosten.

1. Suchen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen** " nach **Speicherkonten** und wählen Sie sie aus.
1. Wählen Sie auf der Seite **Speicherkonten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Seite **Speicherkonto erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Erweitert >**.

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Resource group|**acss-infra-RG**|
   |Speicherkontoname|Ein beliebiger global eindeutiger Name, der zwischen 3 und 24 Zeichen lang ist und aus Buchstaben und Ziffern besteht.|
   |Region|der Name der gleichen Azure-Region, die Sie zuvor in dieser Übung verwendet haben|
   |Leistung|**Standard**|
   |Redundanz|**Georedundanter Speicher (GRS)**|
   |Bei regionaler Verfügbarkeit Lesezugriff auf die Daten bereitstellen|Deaktiviert|

1. Überprüfen Sie auf der **Registerkarte "Erweitert** " die verfügbaren Optionen, übernehmen Sie die Standardwerte, und wählen Sie "Weiter" aus **: Netzwerk >** aus.
1. Führen Sie auf der Registerkarte **Networking** die folgenden Aktionen aus, und wählen Sie dann **Überprüfen + erstellen** aus:

   1. **Öffentlichen Zugriff **über ausgewählte virtuelle Netzwerke und IP-Adressen aktivieren
   1. Stellen Sie im Abschnitt **"Virtuelle Netzwerke"** sicher, dass in der Dropdownliste **"Virtuelles Netzwerkabonnement"** der Name des Azure-Abonnements angezeigt wird, das Sie in dieser Übung verwenden.
   1. Wählen Sie im Abschnitt **"Virtuelle Netzwerke"** in der Dropdownliste **"Virtuelle Netzwerke"**** acss-infra-VNET** aus.
   1. Wählen Sie in der **Dropdownliste "Subnetze** " die **Subnetze "App**", **"db**" und **"dmz** " aus.

1. Warten Sie auf der **Registerkarte "Überprüfen** ", bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **"Erstellen" aus**.

   >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Die Bereitstellung sollte weniger als 1 Minute dauern.

1. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.
1. Wählen Sie auf der Seite "Speicherkonto" im vertikalen Navigationsmenü auf der linken Seite im **Abschnitt "Datenspeicher** " die Option "Container" ** aus**.
1. Wählen Sie **+ Container** aus.
1. Geben Sie im Bereich **"Neuer Container **" im** Textfeld "Name **" sapbits**** ein, und wählen Sie "Erstellen" **aus **.

   >**Hinweis:** Der **Sapbits-Container** hostet die SAP-Installationsmedien.

#### Aufgabe 5: Konfigurieren der Autorisierung der vom Microsoft Entra zugewiesenen verwalteten Identität

In dieser Aufgabe verwenden Sie eine Rollenzuweisung für die rollenbasierte Azure-Zugriffssteuerung (RBAC), um die vom Microsoft Entra zugewiesene verwaltete Identität zu gewähren. Die verwaltete Identität wird verwendet, um den Bereitstellungszugriff auf das Microsoft Azure-Abonnement und das Azure Storage General Purpose v2-Konto auszuführen, das in der vorherigen Aufgabe erstellt wurde.

1. Suchen Sie im Azure-Portal im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen** " nach verwalteten Identitäten ** und wählen Sie sie aus**.
1. Wählen Sie auf der Seite "Verwaltete Identitäten" den **Acss-infra-MI-Eintrag** aus.
1. Wählen Sie auf der **Acss-infra-MI-Seite** im vertikalen Navigationsmenü auf der linken Seite **Azure-Rollenzuweisungen** aus.
1. Wählen Sie auf der **Seite "Azure-Rollenzuweisungen** " + **Rollenzuweisung hinzufügen (Vorschau)** aus.
1. Geben Sie auf der Seite **+ Rollenzuweisung hinzufügen (Vorschau)** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

   |Einstellung|Wert|
   |---|---|
   |Umfang|**Abonnement**|
   |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Rolle|**Azure Center for SAP solutions-Dienstrolle**|

1. Navigieren Sie zurück zur Seite **Azure-Rollenzuweisungen**, und wählen Sie **+ Rollenzuweisung hinzufügen (Vorschau)** aus.
1. Geben Sie auf der Seite **+ Rollenzuweisung hinzufügen (Vorschau)** die folgenden Einstellungen an, und wählen Sie **Speichern** aus:

   |Einstellung|Wert|
   |---|---|
   |Umfang|**Storage**|
   |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
   |Ressource|Der Name des Azure Storage-Kontos, das Sie in der vorherigen Aufgabe erstellt haben|
   |Rolle|**Lese- und Datenzugriff**|

#### Aufgabe 6: Erstellen eines Azure Premium-Dateifreigabekontos

In dieser Aufgabe erstellen Sie ein Azure Premium-Dateifreigabekonto, das zum Implementieren des SAP-Transportverzeichnisses verwendet wird.

1. Suchen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen** " nach **Speicherkonten** und wählen Sie sie aus.
1. Wählen Sie auf der Seite **Speicherkonten** die Option **+ Erstellen** aus.
1. Geben Sie auf der Seite **Speicherkonto erstellen** auf der Registerkarte **Grundeinstellungen** die folgenden Einstellungen an, und wählen Sie **Weiter: Erweitert >**.

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
   |Resource group|**acss-infra-RG**|
   |Speicherkontoname|Ein beliebiger global eindeutiger Name, der zwischen 3 und 24 Zeichen lang ist und aus Buchstaben und Ziffern besteht.|
   |Region|der Name der gleichen Azure-Region, die Sie zuvor in dieser Übung verwendet haben|
   |Leistung|**Premium**|
   |Premium account type (Premium-Kontotyp)|**Dateifreigaben**|
   |Redundanz|**Zonenredundanter Speicher (ZRS)**|

1. Deaktivieren Sie auf der **Registerkarte "Erweitert** " die **Einstellung "Sichere Übertragung für REST-API-Vorgänge** erforderlich", und wählen Sie "Weiter" aus **: Netzwerk >** aus.

   >**Hinweis:** Das NFS-Protokoll unterstützt keine Verschlüsselung und basiert stattdessen auf der Sicherheit auf Netzwerkebene. Diese Einstellung muss deaktiviert sein, damit NFS funktioniert.

1. Führen Sie auf der Registerkarte **Networking** die folgenden Aktionen aus, und wählen Sie dann **Überprüfen + erstellen** aus:

   1. **Öffentlichen Zugriff **über ausgewählte virtuelle Netzwerke und IP-Adressen aktivieren
   1. Stellen Sie im Abschnitt **"Virtuelle Netzwerke"** sicher, dass in der Dropdownliste **"Virtuelles Netzwerkabonnement"** der Name des in dieser Übung verwendeten Azure-Abonnements angezeigt wird.
   1. Wählen Sie im Abschnitt **"Virtuelle Netzwerke"** in der Dropdownliste **"Virtuelle Netzwerke"**** acss-infra-VNET** aus.
   1. Wählen Sie in der **Dropdownliste "Subnetze** " die **Subnetze "App**", **"db**" und **"dmz** " aus.

   >**Hinweis:** Vermeiden Sie im Allgemeinen, den Zugriff auf Ihre internen Ressourcen aus Umkreissubnetzen zuzulassen. In diesem Fall besteht der einzige Grund dafür darin, die Überprüfung dieses Zugriffs später in dieser Übung zuzulassen.

1. Warten Sie auf der **Registerkarte "Überprüfen** ", bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **"Erstellen" aus**.

   >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Die Bereitstellung sollte weniger als 1 Minute dauern.

1. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.
1. Wählen Sie auf der Seite "Speicherkonto" im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Datenspeicher** **Dateifreigaben** aus, und wählen Sie dann **+ Dateifreigabe**aus.
1. Geben Sie auf der **Registerkarte "Grundlagen"** der **Seite "Neue Dateifreigabe** " die folgenden Einstellungen an, und wählen Sie **dann "Überprüfen+ erstellen**" aus:

   |Einstellung|Wert|
   |---|---|
   |Name|**#Trans**|
   |Bereitgestellte Kapazität|**128**|
   |Protokoll|**NFS**|
   |Root-Squash|*Kein Root-Squash**|

1. Warten Sie auf der **Registerkarte "Überprüfen+ Erstellen** ", bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie dann **"Erstellen"** aus.

   >**Hinweis:** Warten Sie, bis die Bereitstellung der Dateifreigabe abgeschlossen ist. Die Bereitstellung sollte nur ein paar Sekunden dauern.

1. Wählen Sie auf der** Seite "Mit dieser NFS-Freigabe von Linux **verbinden" in der** Dropdownliste "Auswählen Ihrer Linux-Verteilung **" in der Dropdownliste für Linux-Distribution **SUSE** aus, und überprüfen Sie die Beispielbefehle, um diese NFS-Freigabe zu mounten.

#### Aufgabe 7: Erstellen und Konfigurieren einer Netzwerksicherheitsgruppe

In dieser Aufgabe erstellen und konfigurieren Sie eine Netzwerksicherheitsgruppe (Network Security Group, NSG), die verwendet wird, um ausgehenden Zugriff von Subnetzen des virtuellen Netzwerks einzuschränken, das die Bereitstellung hostet. Sie können dies erreichen, indem Sie die Verbindung mit dem Internet blockieren, aber explizit Verbindungen mit den folgenden Diensten zulassen:

- SUSE- oder Red Hat-Updateinfrastrukturendpunkte
- Azure Storage
- Azure-Schlüsseltresor
- Microsoft Entra ID
- Azure Resource Manager

>**Hinweis:** Im Allgemeinen sollten Sie die Verwendung von Azure Firewall anstelle von NSGs in Betracht ziehen, um die Netzwerkkonnektivität für Ihre SAP-Bereitstellung zu sichern. In dieser Übung werden beide Optionen behandelt.

1. Suchen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen**" nach Netzwerksicherheitsgruppen ** und wählen Sie sie aus**.
1. Wählen Sie auf der Seite **Netzwerksicherheitsgruppen** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Netzwerksicherheitsgruppe erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
   |Resource group|**acss-infra-RG**|
   |Name|**acss-infra-NSG**|
   |Region|der Name der gleichen Azure-Region, die Sie zuvor in dieser Übung verwendet haben|

1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Die Bereitstellung sollte weniger als 1 Minute dauern.

1. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.

   >**Hinweis:** Standardmäßig ermöglichen die integrierten Regeln von Netzwerksicherheitsgruppen den gesamten ausgehenden Datenverkehr, den gesamten Datenverkehr innerhalb desselben virtuellen Netzwerks sowie den gesamten Datenverkehr zwischen virtuellen Netzwerken mit Peering. Aus Sicherheitsgründen sollten Sie dieses Standardverhalten einschränken. Die vorgeschlagene Konfiguration schränkt die ausgehende Verbindung mit dem Internet und Azure ein. Sie können auch NSG-Regeln verwenden, um die Konnektivität in einem virtuellen Netzwerk einzuschränken.

1. Wählen Sie auf der **Seite acss-infra-NSG** im vertikalen Navigationsmenü auf der linken Seite im **Abschnitt "Einstellungen** " die Option "Ausgehende Sicherheitsregeln **"** aus.
1. Wählen Sie auf der **Seite acss-infra-NSG \| Ausgehende Sicherheitsregeln** + **Hinzufügen**aus.
1. Geben Sie im Bereich**Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Verbindung mit Red Hat-Updateinfrastrukturendpunkten explizit zuzulassen.

   >**Hinweis:** Informationen zur Ermittlung der für RHEL zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints).

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**IP-Adressen**|
   |Ziel-IP-Adressen/CIDR-Bereiche|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Zulassen**|
   |Priorität|**300**|
   |Name|**AllowAnyRHELOutbound**|
   |Beschreibung|**Ausgehende Konnektivität mit RHEL-Updateinfrastrukturendpunkten zulassen**|

1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Konnektivität mit SUSE-Updateinfrastrukturendpunkten explizit zuzulassen.

   >**Hinweis:** Informationen zur Ermittlung der für SUSE zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints).

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**IP-Adressen**|
   |Ziel-IP-Adressen/CIDR-Bereiche|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Zulassen**|
   |Priority|**305**|
   |Name|**AllowAnySUSEOutbound**|
   |Beschreibung|**Ausgehende Konnektivität mit SUSE-Updateinfrastrukturendpunkten zulassen**|

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Verbindung mit Azure Storage explizit zuzulassen.

1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**Diensttag**|
   |Zieldiensttag|**Storage**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Zulassen**|
   |Priority|**400**|
   |Name|**AllowAnyCustomStorageOutbound**|
   |Beschreibung|**Ausgehende Konnektivität mit Azure Storage zulassen**|

   >**Hinweis:** Sie könnten das Diensttag **Storage** durch ein regionsspezifisches Tag ersetzen, z. B. durch **Storage.EastUS**.

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Verbindung mit Azure Key Vault explizit zuzulassen.

1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**Diensttag**|
   |Zieldiensttag|**AzureKeyVault**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Zulassen**|
   |Priority|**500**|
   |Name|**AllowAnyCustomKeyVaultOutbound**|
   |Beschreibung|**Ausgehende Konnektivität mit Azure Key Vault zulassen**|

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Verbindung mit Microsoft Entra ID explizit zuzulassen.

1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**Diensttag**|
   |Zieldiensttag|**AzureActiveDirectory**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Zulassen**|
   |Priority|**600**|
   |Name|**AllowAnyCustomEntraIDOutbound**|
   |Beschreibung|**Ausgehende Konnektivität mit Microsoft Entra ID zulassen**|

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Verbindung mit Azure Resource Manager explizit zuzulassen.

1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**Diensttag**|
   |Zieldiensttag|**AzureResourceManager**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Zulassen**|
   |Priority|**700**|
   |Name|**AllowAnyCustomARMOutbound**|
   |Beschreibung|**Ausgehende Konnektivität mit Azure Resource Manager zulassen**|

   >**Hinweis:** Die letzte Regel sollte hinzugefügt werden, um die Verbindung mit dem Internet explizit zu blockieren.

1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich **Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**Diensttag**|
   |Zieldiensttag|**Internet**|
   |Dienst|**Benutzerdefiniert**|
   |Zielportbereiche|*|
   |Protocol|**Alle**|
   |Aktion|**Deny** (Verweigern)|
   |Priority|**1000**|
   |Name|**DenyAnyCustomInternetOutbound**|
   |Beschreibung|**Ausgehende Verbindung mit dem Internet verweigern**|

   >**Hinweis:** Schließlich müssen Sie die NSG den relevanten Subnetzen des virtuellen Netzwerks zuweisen, die die SAP-Bereitstellung hosten.

1. **Wählen Sie im Bereich "Ausgehende Sicherheitsregel** hinzufügen" im vertikalen Navigationsmenü auf der linken Seite im **Abschnitt "Einstellungen** " die Option **"Subnetze**" aus.
1. Wählen Sie auf der **Seite acss-infra-NSG-\|Subnetze** +Zuordnen **** aus.
1. Wählen Sie im Bereich **Subnetz zuordnen** in der Dropdownliste **Virtuelles Netzwerk** die Option **acss-intra-VNET (acss-infra-RG)** und in der Dropdownliste **Subnetz** die Option **App** aus. Wählen Sie anschließend **OK** aus.
1. **Wählen Sie im **Subnetzbereich** in der Dropdownliste **"Virtuelles Netzwerk"** die Option acss-intra-VNET (acss-infra-RG)**, in der Dropdownliste **"Subnetz" **die Option **"DB"** und dann **"OK"** aus.

#### Aufgabe 8 Erstellen eines virtuellen Azure-Computers

In dieser Aufgabe erstellen Sie einen virtuellen Azure-Computer (VM), der für die SAP-Softwareinstallation als Teil einer Bereitstellung von Azure Center for SAP solutions verwendet wird.

1. Suchen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen**" nach **Virtuellen Computern** und wählen Sie sie aus.
1. Wählen Sie auf der **Seite "Virtuelle Computer** " die Option **"+Erstellen** " aus, und wählen Sie im Dropdownmenü **Azure Virtual Machine** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Erstellen eines virtuellen Computers** die folgenden Einstellungen an, und wählen Sie **Weiter aus: Disks > ** (Behalten Sie für alle anderen Einstellungen die Standardwerte bei):

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
   |Resource group|**acss-infra-RG**|
   |Name des virtuellen Computers|**acss-infra-vm0**|
   |Region|der Name der gleichen Azure-Region, die Sie zuvor in dieser Übung verwendet haben|
   |Verfügbarkeitsoptionen|**Keine Infrastrukturredundanz erforderlich**|
   |Sicherheitstyp|**VMs mit vertrauenswürdigem Start**|
   |Abbildung|**Ubuntu Server 20.04 LTS – x64 Gen2**|
   |VM-Architektur|**x64**|
   |Mit Azure Spot-Rabatt ausführen|deaktiviert|
   |Größe|**Standard_B2ms**|
   |Authentifizierungstyp|**Kennwort**|
   |Username|Beliebiger gültiger Benutzername|
   |Kennwort|beliebiges komplexes Kennwort Ihrer Wahl|
   |Öffentliche Eingangsports|**Keine**|
   
    > **Hinweis:** Stellen Sie sicher, dass Sie sich den von Ihnen angegebenen Benutzernamen und das Kennwort merken. Sie benötigen diese später in diesem Lab.

1. Übernehmen Sie auf der **Registerkarte "Datenträger** " die Standardwerte, und wählen Sie "Weiter" aus **: Netzwerk >** aus.
1. Geben Sie auf der **Registerkarte "Netzwerk** " die folgenden Einstellungen an, und wählen Sie "Weiter" aus **: Verwaltung >** (alle anderen Einstellungen mit ihrem Standardwert belassen):

   |Einstellung|Wert |
   |---|---|
   |Virtuelles Netzwerk|**acss-infra-VNET**|
   |Subnetz|**dmz**|
   |Öffentliche IP-Adresse|**Keine**|
   |NIC-Netzwerksicherheitsgruppe|**Keine**|
   |Löschen Sie die NIC beim Löschen des virtuellen Computers|enabled|
   |Optionen für den Lastenausgleich|**Keine**|

1. Lassen Sie auf der **Registerkarte "Verwaltung** " alle Einstellungen mit ihrem Standardwert, und wählen Sie "Weiter" aus **: Überwachung>**
1. Legen Sie auf der Registerkarte **"Überwachung"** die **Startdiagnose** auf **"Deaktivieren"** fest, und wählen Sie **"Weiter" aus: Erweitert >** (alle anderen Einstellungen mit ihrem Standardwert belassen)
1. Wählen Sie auf der **Registerkarte "Erweitert** " die Option **"Überprüfen und Erstellen** " aus (lassen Sie alle Einstellungen mit ihrem Standardwert).
1. Wählen Sie auf der **Registerkarte "Überprüfen + Erstellen** " des Menüs **"Virtuellen Computer erstellen"** die Option **"Erstellen"** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen wurde. Die Bereitstellung kann ungefähr 3 Minuten dauern.

#### Aufgabe 9 Konfigurieren des virtuellen Azure-Computers

In dieser Aufgabe stellen Sie eine Verbindung mit dem virtuellen Azure-Computer mithilfe von Azure Bastion her und konfigurieren sie für die SAP-Softwareinstallation. 

> **Hinweis:** Bevor Sie diese Aufgabe starten, stellen Sie sicher, dass die Azure Bastion-Bereitstellung abgeschlossen ist.

> **Hinweis:** Stellen Sie sicher, dass Ihr Webbrowser Popupfenster nicht blockiert und deaktivieren Sie in diesem Fall die Popupblockerfunktion.

1. Suchen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im **Textfeld "Suchen**" nach **Virtuellen Computern** und wählen Sie sie aus. 
1. Wählen Sie auf der **Seite "Virtuelle Computer** " den **Eintrag "acss-infra-vm0"** aus.
1. Wählen Sie auf der **Seite acss-infra-vm0** in der Symbolleiste **"Verbinden"** aus, und wählen Sie im Dropdownmenü **"Über Bastion verbinden"** aus.
1. Stellen Sie auf der **Seite acss-infra-vm0 \| Bastion** sicher, dass der **Authentifizierungstyp** auf **"VM-Kennwort"** festgelegt ist, geben Sie in den **Textfeldern "Benutzername** " und **"Kennwort" den Benutzernamen und das Kennwort** ein, den Sie beim Bereitstellen der Azure-VM festgelegt haben, stellen Sie sicher, dass das **Kontrollkästchen "In neuem Browser öffnen"** aktiviert ist, und wählen Sie dann "Verbinden" **aus**.

   > **Hinweis:** Dadurch sollte eine andere Webbrowserfensterregisterkarte geöffnet werden, auf der die Shellsitzung angezeigt wird, die auf der Azure-VM ausgeführt wird.

   > **Hinweis:** Um den Ubuntu-Server für den Upload von SAP-Installationsmedien vorzubereiten, installieren Sie Azure CLI.

1. Führen Sie auf der neu geöffneten Browserregisterkarte in der Shellsitzung den folgenden Befehl aus, um Azure CLI zu installieren:

   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

1. Führen Sie in der Shellsitzung den folgenden Befehl aus, um PIP3 zu installieren:

   ```bash
   sudo apt install python3-pip
   ```

1. Führen Sie in der Shellsitzung den folgenden Befehl aus, um Ansible 2.13.19 zu installieren:

   ```bash
   sudo pip3 install ansible-core==2.13.9
   ```

1. Führen Sie in der Shellsitzung den folgenden Befehl aus, um Ansible Galaxy Collection-Module zu installieren:

   ```bash
   sudo ansible-galaxy collection install ansible.netcommon:==5.0.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.posix:==1.5.1 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.utils:==2.9.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.windows:==1.13.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install community.general:==6.4.0 -p /opt/ansible/collections
   ```

1. Führen Sie in der Shellsitzung den folgenden Befehl aus, um das SAP-Automatisierungsbeispiel-Repository von GitHub zu klonen:

   ```bash
   git clone https://github.com/Azure/SAP-automation-samples.git
   ```

1. Führen Sie in der Shellsitzung den folgenden Befehl aus, um das SAP-Automatisierungs-Repository von GitHub zu klonen:

   ```bash
   git clone https://github.com/Azure/sap-automation.git
   ```

1. Führen Sie in der Shellsitzung den folgenden Befehl aus, um die Sitzung zu beenden:

   ```bash
   logout
   ```

1. Wenn Sie dazu aufgefordert werden, wählen Sie **Schließen**.

#### Aufgabe 10 Entfernen Sie Azure-Ressourcen,

In dieser Aufgabe entfernen Sie alle in dieser Übung bereitgestellten Azure-Ressourcen.

1. Wählen Sie auf dem Laborcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, das **Cloud Shell-Symbol** aus, um den Cloud Shell-Bereich zu öffnen. Wählen Sie **bei Bedarf Bash** aus, um eine Bash-Shellsitzung zu starten. 

   > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal in dem Azure-Abonnement starten, das Sie in dieser Übung verwendet haben, werden Sie aufgefordert, eine Azure-Dateifreigabe zum Speichern von Cloud Shell-Dateien zu erstellen. Akzeptieren Sie in diesem Fall die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Ressourcengruppe **acss-infra-RG** und alle zugehörigen Ressourcen zu löschen.

   ```cli
   az group delete --name 'acss-infra-RG' --no-wait --yes
   ```

   > **Hinweis:** Der Befehl wird asynchron ausgeführt (wie durch den `--nowait` Parameter bestimmt), sodass die Shell-Eingabeaufforderung unmittelbar nach dem Aufrufen angezeigt wird, werden einige Minuten übergeben, bevor die Ressourcengruppe und die zugehörigen Ressourcen tatsächlich entfernt werden.

1. Schließen Sie den Cloud Shell-Bereich.
