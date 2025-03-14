---
lab:
  title: 06b – Übersicht über die Bereitstellung und Wartung von Azure Center for SAP solutions (ACSS)
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Modul AZ 1006: Entwerfen und Implementieren einer Infrastruktur zur Unterstützung von SAP-Workloads in Azure
# 1-tägiges Kurslab AZ-1006: Übersicht über die Bereitstellung und Wartung von Azure Center for SAP solutions (ACSS)

>**Wichtig:** Dieses **Lab wird derzeit nicht unterstützt** (Februar 2024).
    - Detaillierte manuelle Bereitstellungsvoraussetzungen finden Sie unter [Lab 6a](https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab06a-ACSS_One_Day.md)
    - Informationen zur Bereitstellung der ACSS-Demoinfrastruktur finden Sie unter [Lab 6c(https://github.com/MicrosoftLearning/AZ-120-Planning-and-Administering-Microsoft-Azure-for-SAP-Workloads/blob/master/Instructions/AZ-120_Lab06c-ACSS_One_Day.md)].

Geschätzte Dauer: 100 Minuten

Alle Aufgaben in diesem 1-tägigen Kurslab AZ-1006 werden über das Azure-Portal durchgeführt.

## Ziele

Nach Abschluss dieses Labs können Sie Folgendes:

- Implementieren der Mindestvoraussetzungen für die Auswertung der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions
- Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, unter Verwendung von Azure Center for SAP solutions
- Verwalten von SAP-Workloads in Azure zusammen mit Azure Center for SAP solutions

## Anweisungen

### Übung 1: Implementieren der Mindestvoraussetzungen für die Auswertung der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions

Dauer: 30 Minuten

In dieser Übung implementieren Sie die Mindestvoraussetzungen für die Auswertung der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions. Dazu gehören die folgenden Aktivitäten:

>**Wichtig:** Die in dieser Übung implementierten Voraussetzungen sollen *nicht* die bewährten Methoden für die Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions darstellen. Ihr Zweck besteht darin, Zeit, Kosten und Ressourcen zu minimieren, die erforderlich sind, um die Mechanismen der Bereitstellung von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions zu bewerten und Verwaltungs- und Wartungsaufgaben nach der Bereitstellung durchzuführen. Die Implementierung der Voraussetzungen umfasst die folgenden Aktivitäten:

- Erstellen einer benutzerseitig zugewiesenen verwalteten Microsoft Entra-Identität, die von Azure Center for SAP solutions für den Azure Storage-Zugriff während der Bereitstellung verwendet werden soll.
- Gewähren von Zugriff auf das Azure-Abonnement und das Azure Storage-Konto vom Typ „Allgemein v2“ für die benutzerseitig zugewiesene verwaltete Microsoft Entra-Identität, die zum Ausführen der Bereitstellung verwendet wird
- Erstellen des virtuellen Azure-Netzwerks, in dem alle Azure-VMs gehostet werden, die in der Bereitstellung enthalten sind
- Erstellen und Konfigurieren einer Netzwerksicherheitsgruppe (Network Security Group, NSG), die verwendet wird, um ausgehenden Zugriff von Subnetzen des virtuellen Netzwerks einzuschränken, das die Bereitstellung hostet
- Erstellen und Konfigurieren eines NAT-Gateways, das ausgehende Verbindungen von Azure-VMs ermöglicht, die Teil der Bereitstellung sind

Diese Aktivitäten entsprechen den folgenden Aufgaben dieser Übung:

- Aufgabe 1: Erstellen einer benutzerseitig zugewiesenen verwalteten Microsoft Entra-Identität
- Aufgabe 2: Konfigurieren der RBAC-Rollenzuweisungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) in Azure für die benutzerseitig zugewiesene verwaltete Microsoft Entra ID-Identität
- Aufgabe 3: Erstellen des virtuellen Azure-Netzwerks
- Aufgabe 4: Erstellen und Konfigurieren einer Netzwerksicherheitsgruppe
- Aufgabe 5: Erstellen und Konfigurieren eines NAT-Gateways

#### Aufgabe 1: Erstellen einer benutzerseitig zugewiesenen verwalteten Microsoft Entra-Identität

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

1. Warten Sie auf der Registerkarte **Überprüfen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie nicht, bis der Bereitstellungsprozess abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die Bereitstellung sollte nur ein paar Sekunden dauern.

   >**Hinweis:** In einer der anstehenden Aufgaben autorisieren Sie den Zugriff der verwalteten Identität auf das Speicherkonto, das die SAP-Installationsmedien hostet, um die Installation von SAP-Software über Azure Center for SAP solutions zu ermöglichen.

#### Aufgabe 2: Konfigurieren der RBAC-Rollenzuweisungen (Role-Based Access Control, rollenbasierte Zugriffssteuerung) in Azure für das Microsoft Entra ID-Benutzerkonto, das zum Ausführen der Bereitstellung verwendet wird

1. Suchen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Abonnements**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **Abonnements** den Eintrag aus, der das Azure-Abonnement darstellt, das Sie für dieses Lab verwenden werden. 
1. Wählen Sie auf der Seite, auf der die Eigenschaften des Azure-Abonnements angezeigt werden, die Option **Zugriffssteuerung (IAM)** aus.
1. Wählen Sie auf der Seite **Zugriffssteuerung (IAM)** die Option **+ Hinzufügen** und dann im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Liste der **Auftragsfunktionsrollen** nach dem Eintrag **Azure Center for SAP solutions-Dienstrolle**, und wählen Sie die entsprechende Option aus. Wählen Sie anschließend **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Mitglieder** der Seite **Rollenzuweisung hinzufügen** für **Zugriff zuweisen zu** die Option **Verwaltete Identität** aus, und klicken Sie dann auf **+ Mitglieder auswählen**.
1. Geben Sie im Bereich **Verwaltete Identitäten auswählen** die folgenden Einstellungen an:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
   |Verwaltete Identität|**Benutzerseitig zugewiesene verwaltete Identität**|
   |Auswählen|**acss-infra-MI**|
1. Wählen Sie in der Liste der verwalteten Identitäten den Eintrag **acss-infra-MI** aus, und klicken Sie dann auf **Auswählen**.
1. Navigieren Sie zurück zur Registerkarte **Mitglieder**, und wählen Sie **Überprüfen + zuweisen** aus.  
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.
   
   ##### Hinzufügen: „Identität für Azure Center for SAP solutions-Administrator“
   
1. Navigieren Sie zurück zur Seite **Zugriffssteuerung (IAM)**, und wählen Sie **+ Hinzufügen** und dann im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Liste der **Auftragsfunktionsrollen** nach dem Eintrag **Azure Center for SAP solutions-Administrator**, und wählen Sie die entsprechende Option aus. Wählen Sie anschließend **Weiter** aus.
1. Auf der Registerkarte **Mitglieder**
   - Wählen Sie unter **Zugriff zuweisen zu** die Option **Benutzer, Gruppe oder Dienstprinzipal** aus.
   - Klicken Sie auf **+ Mitglieder auswählen**.
1. Geben Sie im Bereich **Mitglieder auswählen** im Textfeld **Auswählen** den Namen des Microsoft Entra ID-Benutzerkontos ein, das Sie für den Zugriff auf das Azure-Abonnement verwendet haben, das Sie für dieses Lab verwenden, wählen Sie es in der Liste der Ergebnisse aus, die Ihrem Eintrag entsprechen, und klicken Sie dann auf **Auswählen**.
1. Navigieren Sie zurück zur Registerkarte **Mitglieder**, und wählen Sie **Überprüfen + zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.
   
   ##### Hinzufügen: „Operator für verwaltete Identität“
   
1. Navigieren Sie zurück zur Seite **Zugriffssteuerung (IAM)**, und wählen Sie **+ Hinzufügen** und dann im Dropdownmenü die Option **Rollenzuweisung hinzufügen** aus.
1. Suchen Sie auf der Registerkarte **Rolle** der Seite **Rollenzuweisung hinzufügen** in der Auflistung **Auftragsfunktionsrollen** nach dem Eintrag **Operator für verwaltete Identität**, und wählen Sie **Weiter** aus.
1. Auf der Registerkarte **Mitglieder**
   - Wählen Sie unter **Zugriff zuweisen zu** die Option **Benutzer, Gruppe oder Dienstprinzipal** aus.
   - Klicken Sie auf **+ Mitglieder auswählen**.
1. Geben Sie im Bereich **Mitglieder auswählen** im Textfeld **Auswählen** den Namen des Microsoft Entra ID-Benutzerkontos ein, das Sie für den Zugriff auf das Azure-Abonnement verwendet haben, das Sie für dieses Lab verwenden, wählen Sie es in der Liste der Ergebnisse aus, die Ihrem Eintrag entsprechen, und klicken Sie dann auf **Auswählen**.
1. Navigieren Sie zurück zur Registerkarte **Mitglieder**, und wählen Sie **Überprüfen + zuweisen** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + Zuweisen** **Überprüfen + Zuweisen** aus.

#### Aufgabe 3: Erstellen des virtuellen Netzwerks

In dieser Aufgabe erstellen Sie das virtuelle Azure-Netzwerk, in dem alle Azure-VMs gehostet werden, die in der Bereitstellung enthalten sind. Darüber hinaus erstellen Sie im virtuellen Netzwerk die folgenden Subnetze:

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

1. Übernehmen Sie auf der Registerkarte **Sicherheit** die Standardeinstellungen, und wählen Sie **Weiter** aus.

   >**Hinweis:** Sie können zu diesem Zeitpunkt sowohl Azure Bastion als auch Azure Firewall bereitstellen, Sie stellen sie jedoch nach der Erstellung des virtuellen Netzwerks separat bereit.

1. Geben Sie auf der Registerkarte **IP-Adressen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |IP-Adressbereich|**/10.0.0.0/16 (65.536 Adressen)**|

1. Wählen Sie in der Liste der Subnetze das Papierkorbsymbol aus, um das **Standardsubnetz** zu löschen.
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

#### Aufgabe 4: Erstellen und Konfigurieren einer Netzwerksicherheitsgruppe

In dieser Aufgabe erstellen und konfigurieren Sie eine Netzwerksicherheitsgruppe (Network Security Group, NSG), die verwendet wird, um ausgehenden Zugriff von Subnetzen des virtuellen Netzwerks einzuschränken, das die Bereitstellung hostet. Sie können dies erreichen, indem Sie die Verbindung mit dem Internet blockieren, aber explizit Verbindungen mit den folgenden Diensten zulassen:

- SUSE- oder Red Hat-Updateinfrastrukturendpunkte
- Azure Storage
- Azure-Schlüsseltresor
- Microsoft Entra ID
- Azure Resource Manager

>**Hinweis:** Im Allgemeinen sollten Sie die Verwendung von Azure Firewall anstelle von NSGs in Betracht ziehen, um die Netzwerkkonnektivität für Ihre SAP-Bereitstellung zu sichern. 

1. Suchen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **Netzwerksicherheitsgruppen**, und wählen Sie das entsprechende Ergebnis aus.
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

1. Wählen Sie auf der Seite **acss-infra-NSG** im Navigationsmenü im Abschnitt **Einstellungen** die Option **Sicherheitsregeln für ausgehenden Datenverkehr** aus.
1. Wählen Sie auf der Seite **acss-infra-NSG \| Sicherheitsregeln für ausgehenden Datenverkehr** die Option **+ Hinzufügen** aus.
1. Geben Sie im Bereich**Ausgangssicherheitsregel hinzufügen** die folgenden Einstellungen an, und wählen Sie **Hinzufügen** aus:

   >**Hinweis:** Die folgende Regel sollte hinzugefügt werden, um die Verbindung mit Red Hat-Updateinfrastrukturendpunkten explizit zuzulassen.

   >**Hinweis:** Informationen zur Ermittlung der für RHEL zu verwendenden IP-Adressen finden Sie unter [Vorbereiten des Netzwerks für die Infrastrukturbereitstellung](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints).

   |Einstellung|Wert|
   |---|---|
   |Quelle|**Alle**|
   |Quellportbereiche|*|
   |Destination|**IP-Adressen**|
   |Ziel-IP-Adressen/CIDR-Bereiche|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198,52.136.197.163,20.225.226.182,52.142.4.99,20.248.180.252,20.24.186.80**|
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

   >**Hinweis:** Schließlich müssen Sie die NSG den relevanten Subnetzen des virtuellen Netzwerks zuweisen, das die SAP-Bereitstellung hostet.

1. Wählen Sie im Bereich **Ausgangssicherheitsregel hinzufügen** im Navigationsmenü im Abschnitt **Einstellungen** die Option **Subnetze** aus.
1. Wählen Sie auf der Seite **acss-infra-NSG \| Subnetze** die Option **+ Zuordnen** aus.
1. Wählen Sie im Bereich **Subnetz zuordnen** in der Dropdownliste **Virtuelles Netzwerk** die Option **acss-intra-VNET (acss-infra-RG)** und in der Dropdownliste **Subnetz** die Option **App** aus. Wählen Sie anschließend **OK** aus.
1. Wählen Sie im Bereich **Subnetz zuordnen** in der Dropdownliste **Virtuelles Netzwerk** die Option **acss-intra-VNET (acss-infra-RG)** und in der Dropdownliste **Subnetz** die Option **db** aus. Wählen Sie anschließend **OK** aus.

#### Aufgabe 5: Erstellen und Konfigurieren eines NAT-Gateways

In dieser Aufgabe erstellen und konfigurieren Sie ein NAT-Gateway (Network Address Translation, Netzwerkadressenübersetzung), mit dem in der Bereitstellung enthaltene Azure-VMs externe Dienste wie SUSE- und RHEL-Endpunkte erreichen können.

1. Suchen Sie auf dem Labcomputer im Webbrowserfenster, in dem das Azure-Portal angezeigt wird, im Textfeld **Suchen** nach **NAT-Gateways**, und wählen Sie die entsprechende Option aus.
1. Wählen Sie auf der Seite **NAT-Gateways** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Gateway für die Netzwerkadressübersetzung (NAT) erstellen** die folgenden Einstellungen an, und wählen Sie dann **Weiter: Ausgehende IP-Adresse >** aus:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
   |Resource group|**acss-infra-RG**|
   |Name des NAT-Gateways|**acss-infra-NATGW**|
   |Region|der Name der gleichen Azure-Region, die Sie zuvor in dieser Übung verwendet haben|
   |Verfügbarkeitszone|**Keine Zone**|
   |TCP-Leerlauftimeout (Minuten)|**4**|

1. Wählen Sie auf der Registerkarte **Ausgehende IP-Adresse** unter der Dropdownliste **Öffentliche IP-Adressen** den Link **Neue öffentliche IP-Adresse erstellen** aus. Geben Sie im Bereich **Öffentliche IP-Adresse hinzufügen** im Textfeld **Name** den Namen **acss-infra-NATWG-PIP** ein, und wählen Sie **OK** aus.
1. Wählen Sie auf der Registerkarte **Ausgehende IP-Adresse** die Option **Weiter: Subnetz >** aus.
1. Wählen Sie auf der Registerkarte **Subnetz** in der Dropdownliste **Virtuelles Netzwerk** die Option **acss-infra-VNET** aus, aktivieren Sie in der Liste der Subnetze das Kontrollkästchen neben den Einträgen **App** und **db**, und wählen Sie dann **Überprüfen + Erstellen** aus:
1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Die Bereitstellung sollte ungefähr 1 Minuten dauern.

### Übung 2: Bereitstellen der Infrastruktur, die SAP-Workloads in Azure hosten soll, unter Verwendung von Azure Center for SAP solutions

Duration (Dauer): 40 Minuten

In dieser Übung führen Sie die Bereitstellung von Azure Center for SAP solutions durch. Dazu gehören die folgenden Aktivitäten:

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
   |Resource group|Der Name einer **neuen** Ressourcengruppe: **acss-vi-RG**|
   |Name (SID)|**VI1**|
   |Region|der Name der Azure-Region, in der die ACSS-registrierte SAP-Bereitstellung gehostet wird oder eine andere Region in derselben Geografie|
   |Umgebungstyp|**Nicht-Produktion**|
   |SAP-Produkt|**S/4HANA**|
   |Datenbank|**HANA**|
   |HANA-Skalierungsmethode|**Hochskalieren (empfohlen)**|
   |Bereitstellungstyp|**Verteilt mit Hochverfügbarkeit**|
   |Computeverfügbarkeit|**99,95 (Verfügbarkeitsgruppe)**|
   |Virtuelles Netzwerk|**acss-infra-VNET**|
   |Anwendungssubnetz|**App (10.0.2.0/24)**|
   |Datenbanksubnetz|**db (10.0.3.0/24)**|
   |Optionen für Anwendungsbetriebssystemimages|**Verwenden eines Marketplace-Images**|
   |Anwendungsbetriebssystemimage|**Red Hat Enterprise Linux 8.4 für SAP-Anwendungen – x64 Gen2 neueste Version**|
   |Optionen für Datenbankbetriebssystemimages|**Verwenden eines Marketplace-Images**|
   |Datenbankbetriebssystemimage|**Red Hat Enterprise Linux 8.4 für SAP-Anwendungen – x64 Gen2 neueste Version**|
   |SAP-Transportoption|**SAP-Transportverzeichnis nicht einschließen**|
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

   >**Hinweis:** Passen Sie bei Bedarf die empfohlenen Größen an, indem Sie den Link **Alle Größen anzeigen** für jede Gruppe von VMs festlegen und eine alternative Größe auswählen. Standardmäßig führt der verteilte Bereitstellungstyp mit Hochverfügbarkeit sowie mit dem oben angegebenen SAPS-Wert für die Logikschicht und der oben angegebenen Datenbankspeichergröße zu den folgenden Mindestempfehlungen für die VM-SKU:
   - 2 VMs vom Typ „Standard_D4ds_v5“ für die ASCS-VMs (jeweils 4 vCPUs und 16 GiB Arbeitsspeicher)
   - 2 VMs vom Typ „Standard_D4ds_v5“ für die Anwendungs-VMs (jeweils 4 vCPUs und 16 GiB Arbeitsspeicher)
   - 2 VMs vom Typ „Standard_E16ds_v5“ für die Datenbank-VMs (jeweils 16 vCPUs und 128 GiB Arbeitsspeicher)

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

   >**Hinweis:** Nach der Bereitstellung können Sie die SAP-Software mithilfe von Azure Center for SAP solutions installieren. In diesem Lab erkunden Sie die Funktionen von Azure Center for SAP solutions, ohne SAP-Software zu installieren.

### Übung 3: Verwalten von SAP-Workloads in Azure mithilfe von Azure Center for SAP solutions

Duration (Dauer): 60 Minuten

In dieser Übung überprüfen Sie die Verwaltung und Überwachung von SAP-Workloads nach der Bereitstellung mithilfe von Azure Center for SAP solutions. Dazu gehören die folgenden Aktivitäten:

- Implementieren der Voraussetzungen für die Sicherung von SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 
- Implementieren der Voraussetzungen für die Notfallwiederherstellung von SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 
- Überprüfen der verfügbaren Überwachungsoptionen für SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 
- Löschen aller in diesem Lab bereitgestellten Azure-Ressourcen.

Diese Aktivitäten entsprechen den folgenden Aufgaben dieser Übung:

- Aufgabe 1: Implementieren der Voraussetzungen für die Sicherung von SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 
- Aufgabe 2: Implementieren der Voraussetzungen für die Notfallwiederherstellung von SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 
- Aufgabe 3: Überprüfen der Überwachungsoptionen für SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 
- Aufgabe 4: Löschen der in diesem Lab bereitgestellten Azure-Ressourcen

#### Aufgabe 1: Implementieren der Voraussetzungen für die Sicherung von SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden

>**Hinweis:** Wenn Sie Azure Backup auf der VIS-Ressourcenebene in Azure Center for SAP solutions konfigurieren, können Sie in einem Schritt die Sicherung für Azure-VMs, welche die Datenbank, den Anwendungsserver und die SAP Central Services-Instanz hosten, und für HANA DB aktivieren. Für die HANA DB-Sicherung führt Azure Center for SAP solutions automatisch das Sicherungsvorregistrierungsskript aus.

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
1. Übernehmen Sie auf der Registerkarte **Netzwerk** die Standardoption **Öffentlichen Zugriff aus allen Netzwerken zulassen**, und wählen Sie dann **Überprüfen + Erstellen** aus
1. Warten Sie auf der Registerkarte **Überprüfen+ Erstellen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Die Bereitstellung kann ungefähr 2 Minuten dauern.

1. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.
1. Wählen Sie auf der Seite **acss-backup-RSV** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Verwalten** die Option **Sicherungsrichtlinien** aus.
1. Wählen Sie auf der Seite **acss-backup-RSV \| Sicherungsrichtlinien** die Option **+ Hinzufügen** aus.
1. Überprüfen Sie auf der Seite **Richtlinientyp auswählen** die verfügbaren Richtlinientypen.

   >**Hinweis:** Sie konfigurieren zunächst die Sicherungsrichtlinie für Azure-VMs, die die Datenbank, die Anwendungsserver und die SAP Central Services-Instanz hosten.

1. Wählen Sie auf der Seite **Richtlinientyp auswählen** die Option **Azure-VM** aus.
1. Sehen Sie sich auf der Seite **Richtlinie erstellen** die Unterschiede zwischen den Richtlinienuntertypen **Standard** und **Erweitert** an, und wählen Sie **Erweitert** aus.

   >**Hinweis:** Sie sollten bei der Wahl die Resilienzanforderungen berücksichtigen. Genau so sollten Sie die folgenden aufgeführten Werte entsprechend Ihren Anforderungen anpassen.

1. Geben Sie im Textfeld **Richtlinienname** den Namen **acss-vm-enhanced-backup-policy** ein.
1. Legen Sie die Häufigkeit des Sicherungszeitplans auf **Stündlich**, die Startzeit auf **18 Uhr**, den Zeitplan auf **Alle 6 Stunden**, die Dauer auf **12 Stunden** und die Zeitzone auf die lokale Zeitzone fest.
1. Legen Sie den Wert **Momentaufnahme(n) zur sofortigen Wiederherstellung beibehalten für** auf **5** Tage fest.
1. Legen Sie **Aufbewahrung der täglichen Sicherungspunkte** auf **60** Tage fest.
1. Legen Sie den Aufbewahrungszeitraum für wöchentliche, monatliche und jährliche Sicherungspunkte nach Bedarf fest.

   >**Hinweis:** Zum Aktivieren von Tiering müssen Sie monatliche oder jährliche Sicherungspunkte aktivieren.

   >**Hinweis:** Sie haben auch die Möglichkeit, die Namenskonvention der von Azure Backup automatisch generierten Ressourcengruppe festzulegen.

1. Klicken Sie auf **Erstellen**.
1. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.
1. Wählen Sie auf der Seite **acss-backup-RSV** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Verwalten** die Option **Sicherungsrichtlinien** aus.
1. Wählen Sie auf der Seite **acss-backup-RSV \| Sicherungsrichtlinien** die Option **+ Hinzufügen** aus.

   >**Hinweis:** Als Nächstes konfigurieren Sie die Sicherungsrichtlinien für HANA DB. Die hier vorgestellte Konfiguration befolgt die im MS Learn-Artikel [Sichern von Momentaufnahmen von SAP HANA-Datenbankinstanzen auf Azure-VMs](https://learn.microsoft.com/en-us/azure/backup/sap-hana-database-instances-backup) beschriebene Anleitung.

1. Wählen Sie auf der Seite **Richtlinientyp auswählen** die Option **SAP HANA auf einem virtuellen Azure-Computer (Datenbank über Backint)** aus.
1. Geben Sie auf der Seite **Richtlinie erstellen** im Textfeld **Richtlinienname** den Namen **acss-hanadb-backint-backup-policy** ein.
1. Übernehmen Sie die Standardeinstellungen für die vollständigen Sicherungen und die Protokollsicherungen. Lassen Sie die differenziellen und inkrementellen Sicherungen deaktiviert.

   >**Hinweis:** Sie haben die Möglichkeit, die HANA-Sicherungskomprimierung zu aktivieren und berechtigte Wiederherstellungspunkte in das Tresorarchiv zu verschieben. 

1. Klicken Sie auf **Erstellen**.

1. Klicken Sie auf der Seite **Ihre Bereitstellung wurde abgeschlossen** auf **Zu Ressource wechseln**.
1. Wählen Sie auf der Seite **acss-backup-RSV** im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Verwalten** die Option **Sicherungsrichtlinien** aus.
1. Wählen Sie auf der Seite **acss-backup-RSV \| Sicherungsrichtlinien** die Option **+ Hinzufügen** aus.
1. Wählen Sie auf der Seite **Richtlinientyp auswählen** die Option **SAP HANA auf einem virtuellen Azure-Computer (Datenbankinstanz über Momentaufnahme)** aus.
1. Geben Sie auf der Seite **Richtlinie erstellen** im Textfeld **Richtlinienname** den Namen **acss-hanadb-snapshot-backup-policy** ein.
1. Legen Sie die Häufigkeit der Momentaufnahmesicherung auf 13:30 Uhr Ihrer aktuellen Zeitzone fest.
1. Konfigurieren Sie die Einstellung so, dass Momentaufnahmen zur sofortigen Wiederherstellung **5** Tage beibehalten werden.
1. Behalten Sie die Standardauswahl der Momentaufnahmen-Ressourcengruppe bei, die auf **acss-mgmt-RG** festgelegt sein sollte.
1. Wählen Sie im Abschnitt **Verwaltete Identität** die Option **Verwaltete Identität erstellen** aus. Dadurch wird automatisch eine weitere Registerkarte im Webbrowserfenster geöffnet, auf der die Seite **Verwaltete Identität** im Azure-Portal angezeigt wird.
1. Wählen Sie auf der Seite **Verwaltete Identität** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundlagen** der Seite **Benutzerseitig zugewiesene verwaltete Identität erstellen** die folgenden Einstellungen an, und wählen Sie dann **Überprüfen + Erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Abonnement|Name des Azure-Abonnements, das in diesem Lab verwendet wird|
   |Resource group|**acss-mgmt-RG**|
   |Region|der Name der Azure-Region, in der die ACSS-Bereitstellung gehostet wird|
   |Name|**acss-mgmt-MI**|

1. Warten Sie auf der Registerkarte **Überprüfen**, bis der Überprüfungsprozess abgeschlossen ist, und wählen Sie **Erstellen** aus.

   >**Hinweis:** Warten Sie auf den Abschluss des Bereitstellungsvorgangs. Die Bereitstellung sollte nur ein paar Sekunden dauern.

1. Schließen Sie die aktuelle Browserregisterkarte, und wechseln Sie zurück zu der Registerkarte, auf der die Seite **Richtlinie erstellen** angezeigt wird.
1. Wählen Sie im Abschnitt **Verwaltete Identität** in der Dropdownliste **Ressourcengruppe** die Option **acss-mgmt-RG** und in der Dropdownliste **Verwaltete Identität** die Option **acss-mgmt-MI** aus.
1. Klicken Sie auf **Erstellen**.

   >**Hinweis:** Beim Konfigurieren der Sicherung auf VIS-Ebene in der Schnittstelle für Azure Center for SAP solutions können Sie den vorhandenen Tresor und seine Richtlinien nutzen.

   >**Hinweis:** Sobald die Sicherung auf VIS-Ebene konfiguriert ist, können Sie den Status von Sicherungsaufträgen von Azure-VMs und HANA DB über die VIS-Schnittstelle im Azure-Portal überwachen.

#### Aufgabe 2: Implementieren der Voraussetzungen für die Notfallwiederherstellung von SAP-Workloads, die von Azure Center for SAP solutions verwaltet werden 

>**Hinweis:** Azure Center for SAP solutions ist zwar ein zonenredundanter Dienst, es gibt jedoch kein von Microsoft initiiertes Failover im Falle eines Regionsausfalls. Um dieses Szenario nachzubessern, sollten Sie die Notfallwiederherstellung für SAP-Systeme konfigurieren, die mithilfe von Azure Center for SAP solutions bereitgestellt werden. Befolgen Sie dazu die unter [Übersicht über die Notfallwiederherstellung und Infrastrukturrichtlinien für SAP-Workloads](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide) beschriebene Anleitung, die die Verwendung von Azure Site Recovery (ASR) umfasst. In dieser Aufgabe durchlaufen Sie den Prozess der Implementierung einer ASR-basierten Notfallwiederherstellungslösung, die auf dieser Anleitung basiert.

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

   >**Hinweis:** Sie haben auch die Möglichkeit, eine Kapazitätsreservierung zu konfigurieren.

   >**Wichtig:** Beachten Sie, dass sich die IP-Adressräume zwischen dem virtuellen Netzwerk in der primären und der sekundären Region unterscheiden. Dies ist beabsichtigt, da es die Verbindung zwischen den beiden virtuellen Netzwerken ermöglicht. Das ist erforderlich, um die Replikation zwischen Datenbankservern zu konfigurieren, die in den beiden Regionen gehostet werden. Diese Verbindung kann mithilfe des Peerings virtueller Netzwerke hergestellt werden.

1. Wählen Sie auf der Registerkarte **Replikationseinstellungen** der Seite **Replikation aktivieren** die Option **Weiter** aus.
1. Führen Sie auf der Registerkarte **Verwalten** auf der Seite **Replikation aktivieren** die folgenden Aktionen aus:

   1. Wählen Sie in der Dropdownliste **Replikationsrichtlinie** die Option **Neu erstellen** aus.
   1. Geben Sie im Bereich **Replikationsrichtlinie erstellen** im Textfeld **Name** den Namen **2-day-retention-policy** und im Textfeld **Aufbewahrungsdauer (in Tagen)** den Wert **2** ein. Wählen Sie anschließend **OK** aus.
   1. Sie haben die Möglichkeit, Replikationsgruppen zu erstellen. Diese Option gilt nicht für unseren Anwendungsfall.
   1. Übernehmen Sie für die Option **Updateeinstellungen** unter **Erweiterungseinstellungen** die Konfiguration **Verwaltung durch ASR zulassen**.
   1. Übernehmen Sie den automatisch zugewiesenen Standardnamen des **Automatisierungskontos**.

1. Wählen Sie auf der Registerkarte **Verwalten** auf der Seite **Replikation aktivieren** die Option **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen** der Seite **Replikation aktivieren** die Option **Replikat aktivieren** aus.

   >**Hinweis:** Sie durchlaufen jetzt schrittweise die Aktivierung der Replikation für die beiden verbleibenden Azure-VMs, die die Datenbankserver hosten.

1. Navigieren Sie zurück zur Seite **acss-dr-RSV \| Site Recovery**, und wählen Sie im Abschnitt **Azure-VMs** die Option **1. Aktivieren der Replikation**. 
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

   >**Hinweis:** Sie müssen die Replikation für die beiden verbleibenden Azure-VMs separat konfigurieren, da sie mit einem anderen Subnetz verbunden sind.

1. Führen Sie auf der Registerkarte **Replikationseinstellungen** die folgenden Aktionen aus:

   1. Wählen Sie bei Bedarf in der Dropdownliste **Zielspeicherort** die Azure-Region aus, in der Sie den Recovery Services-Tresor **acss-dr-RSV** erstellt haben.
   1. Stellen Sie sicher, dass der Name des in diesem Lab verwendeten Azure-Abonnements in der Dropdownliste **Zielabonnement** angezeigt wird.
   1. Wählen Sie in der Dropdownliste **Zielressourcengruppe** die Option **acss-dr-RG** aus.
   1. Wählen Sie in der Dropdownliste **Virtuelles Failovernetzwerk** die Option **CONTOSO-VNET** aus.
   1. Wählen Sie in der Dropdownliste **Failoversubnetz** die Option **App (10.10.0.0/24)** aus.
   1. Wählen Sie im Abschnitt **Speicher** den Link **Speicherkonfiguration anzeigen/bearbeiten** aus.
   1. Überprüfen Sie auf der Seite **Zieleinstellungen anpassen** die resultierende Konfiguration, nehmen Sie aber keine Änderungen vor, und wählen Sie **Abbrechen** aus.
   1. Wählen Sie im Abschnitt **Verfügbarkeitsoptionen** den Link **Verfügbarkeitsoptionen anzeigen/bearbeiten** aus.
   1. Auf der Seite **Verfügbarkeitsoptionen** haben Sie die Möglichkeit, Näherungsplatzierungsgruppen für Zielressourcen zu implementieren. Nehmen Sie aber keine Änderungen vor, und wählen Sie **Abbrechen** aus.

   >**Hinweis:** Sie haben auch die Möglichkeit, eine Kapazitätsreservierung zu konfigurieren.

1. Wählen Sie auf der Registerkarte **Replikationseinstellungen** der Seite **Replikation aktivieren** die Option **Weiter** aus.
1. Führen Sie auf der Registerkarte **Verwalten** auf der Seite **Replikation aktivieren** die folgenden Aktionen aus:

   1. Wählen Sie in der Dropdownliste **Replikationsrichtlinie** die Option **Neu erstellen** aus.
   1. Geben Sie im Bereich **Replikationsrichtlinie erstellen** im Textfeld **Name** den Namen **2-day-retention-policy** und im Textfeld **Aufbewahrungsdauer (in Tagen)** den Wert **2** ein. Wählen Sie anschließend **OK** aus.
   1. Sie haben die Möglichkeit, Replikationsgruppen zu erstellen. Diese Option gilt nicht für unseren Anwendungsfall.
   1. Übernehmen Sie für die Option **Updateeinstellungen** unter **Erweiterungseinstellungen** die Konfiguration **Verwaltung durch ASR zulassen**.
   1. Übernehmen Sie den automatisch zugewiesenen Standardnamen des **Automatisierungskontos**.

1. Wählen Sie auf der Registerkarte **Verwalten** auf der Seite **Replikation aktivieren** die Option **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen** der Seite **Replikation aktivieren** die Option **Replikat aktivieren** aus.

   >**Hinweis:** Die Erstreplikation kann eine erhebliche Zeit in Anspruch nehmen. In Anbetracht der begrenzten Zeit, die für dieses Lab zur Verfügung steht, wenden Sie sich an den Kursleiter oder die Kursleiterin, wenn Sie zusätzliche Schritte im Rahmen dieser Aufgabe durchführen müssen. Wenn keine spezifischen Anweisungen vorhanden sind, fahren Sie direkt mit der nächsten Aufgabe fort.

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
