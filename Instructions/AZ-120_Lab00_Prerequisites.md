---
lab:
  title: 00 – Voraussetzungen des Labs
  module: Module 00 - Lab prerequisites
---

# AZ 120: Voraussetzungen des Labs

## vCPU-Kernanforderungen

> **Wichtig:** Die vCPU-Kernanforderungen hängen von den Labs ab, die Sie implementieren möchten.

- Zum Abschließen von Lab 04b – Implementieren der SAP-Architektur auf Azure-VMs unter Windows benötigen Sie ein Microsoft Azure-Abonnement mit mindestens 28 vCPU in der Azure-Region, die Verfügbarkeitszonen unterstützt, in denen sich die in diesem Lab bereitgestellten Azure-VMs befinden.

    - 4 x Standard_DS1_v2 (jeweils 1 vCPUs) = 4
    - 6 x Standard_D4s_v3 (jeweils 4 vCPUs) = 24

    > **Hinweis:** Erwägen der Verwendung der Regionen **USA, Osten** oder **USA, Osten 2** für die Bereitstellung Ihrer Ressourcen

    > **Hinweis:** Informationen zur Identifizierung der Azure-Regionen, die Verfügbarkeitszonen unterstützen, finden Sie unter <https://docs.microsoft.com/en-us/azure/availability-zones/az-overview>

- Um Lab 05 abzuschließen – Automatisieren Sie die Bereitstellung mithilfe von Azure Center für SAP-Lösungen, benötigen Sie ein Microsoft Azure-Abonnement mit der folgenden vCPU-Verfügbarkeit in der Azure-Region, in der sich die in diesem Lab bereitgestellten Azure-VMs befinden.

    - 4 x Standard_E4ds_v4 (jeweils 4 vCPUs) oder 4 X Standard_D4ds_v4 (jeweils 4 vCPUs) = 8
    - 2 x Standard_M64ms (jeweils 64 vCPUs) = 128

>**Hinweis:** Um die vCPU- und Speicheranforderungen zu minimieren, können Sie Standard_M32ts (32 vCPUs und 192 GiB des Arbeitsspeichers) anstelle von Standard_M64m verwenden.

>**Hinweis:** Während die vCPU-Anforderungen für die ersten drei Labs dieses Kurses niedriger sind, empfehlen wir, dass Sie eine Erhöhung der Kontingente anfordern, um Anforderungen für alle Labs zu erfüllen, da der Prozess der Erhöhung der Kontingente einige Zeit in Anspruch nehmen kann (auch wenn Kontingenterhöhungsanforderungen normalerweise am selben Arbeitstag abgeschlossen werden).

## Vor dem Praxislab

Zeitraum: 30 Minuten

### Aufgabe 1: Überprüfen Sie eine ausreichende Anzahl von vCPU-Kernen für das Lab Implementieren Sie die SAP-Architektur auf Azure-VMs unter Windows

1. Starten Sie auf dem Lab-Computer einen Webbrowser und navigieren Sie zum Azure-Portal unter `https://portal.azure.com`.
1. Starten Sie im Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

    > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal im aktuellen Azure-Abonnement starten, werden Sie aufgefordert, eine Azure-Dateifreigabe zu erstellen, um Cloud Shell-Dateien zu speichern. Akzeptieren Sie in diesem Falls die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Azure-Portal im Bereich **Cloud Shell** an der PowerShell-Eingabeaufforderung Folgendes aus: Dabei legt `<Azure_region>` die Ziel-Azure-Region fest, die Sie für dieses Lab verwenden möchten (z. B. `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **Hinweis:** Um die Namen der Azure-Regionen zu ermitteln, führen Sie in der **Cloud Shell** an der Bash-Eingabeaufforderung `(Get-AzLocation).Location` aus
   
1. Überprüfen Sie den aktuellen Wert und die Grenzwerteinträge in der Ausgabe der Befehle, die im vorherigen Schritt ausgeführt werden, und stellen Sie sicher, dass Sie über eine ausreichende Anzahl von vCPUs in der Azure-Zielregion verfügen.
1. Wenn die Anzahl der vCPUs nicht ausreicht, navigieren Sie im Azure-Portal zurück zum Abonnementblatt, und klicken Sie auf **Verbrauch + Kontingente**. 
1. Klicken Sie auf dem Blatt **Verbrauch + Kontingente** des Abonnements auf **Erhöhung anfordern**.
1. Geben Sie auf dem Blatt **Problembeschreibung** Folgendes an:

    -   Problemtyp: **Grenzwerte für Dienste und Abonnements (Kontingente)**
    -   Abonnement: Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden werden
    -   Kontingenttyp: **Abonnementgrenzen für Compute/VM (Kerne/vCPUs) erhöhen**

1. Klicken Sie auf **Kontingent verwalten**.
1. Verwenden Sie das Dropdownmenü „Speicherort“, um die Ergebnisse nach der Azure-Region zu filtern, die Sie verwenden möchten. Es wird empfohlen, **USA, Osten** oder **USA, Osten2** zu verwenden.
1. Suchen Sie den **Standard-DSv3-Familien-vCPUs**-Kontingenttyp und wählen Sie den Bearbeitungsstift aus.
1. Geben Sie im Feld „Neuer Grenzwert **40**an und klicken Sie auf **Speichern und Weiter**.
1. Suchen Sie den Kontingenttyp **Gesamtregionale vCPUs** und wählen Sie den Bearbeitungsstift aus.
1. Geben Sie im Feld „Neuer Grenzwert **40**an und klicken Sie auf **Speichern und Weiter**.

   > **Hinweis:** Die Anforderung zur Erhöhung des Kontingents sollte automatisch genehmigt werden. Wenn die Anforderung verweigert wird, öffnen Sie ein Supportticket, um die Kontingenterhöhung anzufordern.

### Aufgabe 2: Überprüfen der ausreichenden Anzahl von vCPU-Kernen für das Lab „Automate-Bereitstellung“ mithilfe von Azure Center for SAP solutions

> **Hinweis:** Um dieses Lab abzuschließen (wie beschrieben), benötigen Sie ein Microsoft Azure-Abonnement mit den vCPU-Kontingenten, welche die Bereitstellung der folgenden virtuellen Computer ermöglichen:

- 2 x Standard_E4ds_v4 (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher) oder 2 X Standard_D4ds_v4 (4 vCPUs und jeweils 16 GiB Arbeitsspeicher) VMs für die ASCS-Ebene
- 2 x Standard_E4ds_v4 (jeweils 4 vCPUs und 32 GiB Arbeitsspeicher) oder 2 X Standard_D4ds_v4 (4 vCPUs und jeweils 16 GiB Arbeitsspeicher) VMs für die Logikschicht 
- 2 x Standard_M64ms (jeweils 64 vCPUs und 1750 GiB Arbeitsspeicher) VMs für die Datenschicht

1. Starten Sie auf dem Lab-Computer einen Webbrowser und navigieren Sie zum Azure-Portal unter `https://portal.azure.com`.
1. Wählen Sie im Azure-Portal das Symbol **Cloud Shell** aus und starten Sie eine PowerShell-Sitzung in Cloud Shell. 
1. Führen Sie im Azure-Portal im Bereich **Cloud Shell** an der PowerShell-Eingabeaufforderung Folgendes aus: Dabei legt `<Azure_region>` die Azure-Region fest, in der Sie Ressourcen in diesem Lab bereitstellen möchten (z. B. `eastus`):

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **Hinweis:** Um die Namen der Azure-Regionen zu ermitteln, führen Sie in der **Cloud Shell** an der Bash-Eingabeaufforderung `(Get-AzLocation).Location` aus

1. Überprüfen Sie die Ausgabe, um die aktuelle vCPU-Verwendung und den vCPU-Grenzwert zu identifizieren. Stellen Sie sicher, dass der Unterschied zwischen ihnen ausreicht, um vCPUs von Azure-VMs aufzunehmen, die Ihnen in diesem Lab bereitgestellt werden. Berücksichtigen Sie sowohl die VM-familienspezifische als auch die gesamte regionale vCPU-Zahl. 
1. Wenn die Anzahl der vCPUs nicht ausreicht, schließen Sie den Cloud Shell-Bereich im Azure-Portal im Textfeld **Suchen** und wählen Sie **Kontingente** aus.
1. Wählen Sie auf der Seite **Kontingente** die Option **Compute** aus.
1. Verwenden Sie auf der Seite **Kontingente\| Compute** den **Regionenfilter**, um die Azure-Region auszuwählen, in der Sie Ressourcen in diesem Lab bereitstellen möchten.
1. Suchen Sie in der Spalte **Kontingentname** den VM-SKU-Namen, der eine Kontingenterhöhung erfordert, und wählen Sie ihn aus. 
1. Überprüfen Sie in derselben Zeile den Eintrag in der Spalte **Anpassbar**. Der nächste Schritt hängt davon ab, ob die Spalte den Eintrag **Ja** oder **Nein** enthält.

   - Wenn der Eintrag auf **Ja** festgelegt ist, wählen Sie das Symbol **Anpassung anfordern** unter der Option **Neue Kontingentanforderung** im Textfeld **Neue Kontingentgrenze** aus, geben Sie die neue Kontingentgrenze ein und wählen Sie dann **Absenden** aus.
   - Wenn der Eintrag auf **Nein** festgelegt ist, wählen Sie das Symbol **Zugriff anfordern oder Empfehlungen erhalten** und dann im Bereich **Kontingentempfehlungen** die Option **Support kontaktieren** und dann **Weiter** aus. 

1. Geben Sie auf der Registerkarte **Problembeschreibung** der Seite **Neue Supportanfrage** die folgenden Einstellungen an, und wählen Sie dann **Weiter** aus:

    |Einstellung|Wert|
    |---|---|
    |Worauf bezieht sich Ihr Problem?|**Azure-Dienste**|
    |Problemtyp|**Grenzwerte für Dienste und Abonnements (Kontingente)**|
    |Abonnement|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden|
    |Kontingenttyp|**Abonnementgrenzen für Compute/VM (Kerne/vCPUs) erhöhen**|

1. Wählen Sie auf der Registerkarte **Weitere Details** die Option **Details eingeben** aus.
1. Wählen Sie auf der Registerkarte **Kontingentdetails** in der Dropdownliste **Bereitstellungsmodell** die Option **Ressourcen-Manager**, in der Dropdownliste **Speicherorte** wählen Sie die Ziel-Azure-Region aus, unter der Dropdownliste **Kontingente** wählen Sie die Azure-VM-Serie aus, für die Sie die Kontingentgrenzen erhöhen müssen, geben Sie im Textfeld **Neue Grenze** die neue Kontingentgrenze ein und wählen Sie dann **Speichern und Fortsetzen** aus.
1. Zurück auf der Registerkarte **Weitere Details** wählen Sie auf der Registerkarte **Erweiterte Diagnoseinformationen** die Option **Ja (Empfohlen)** aus.
1. Wählen Sie im Abschnitt **Supportmethode** entweder **E-Mail** oder **Telefon** als bevorzugte Kontaktmethode aus und wählen Sie dann **Weiter** aus.
1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** die Option **Erstellen** aus.

    > **Hinweis:** Warten Sie, bis die Anforderung zum Erhöhen von Kontingentbeschränkungen erfolgreich abgeschlossen wurde, bevor Sie die Bereitstellung von Lab Automate mithilfe von Azure Center für SAP-Lösungen starten.
