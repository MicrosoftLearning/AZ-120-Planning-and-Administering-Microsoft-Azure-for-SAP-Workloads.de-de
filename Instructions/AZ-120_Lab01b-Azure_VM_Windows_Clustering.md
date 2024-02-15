---
lab:
  title: '01b: Implementieren von Windows-Clustering auf Azure-VMs'
  module: Module 01 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120, Modul 1: Erkunden der Grundlagen von IaaS für SAP in Azure
# Lab 1b: Implementieren von Windows-Clustering auf Azure-VMs

Geschätzte Dauer: 120 Minuten

Alle Aufgaben in diesem Lab werden über das Azure-Portal ausgeführt (einschließlich der PowerShell Cloud Shell-Sitzung).  

   > **Hinweis:** Wenn Sie Cloud Shell nicht verwenden, muss auf dem virtuellen Labcomputer das PowerShell-Modul „Az“ installiert sein[**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Labdateien: keine

## Szenario
  
Als Vorbereitung für die Bereitstellung von SAP NetWeaver in Azure mit SQL Server als Datenbankverwaltungssystem möchte die Adatum Corporation den Prozess der Implementierung von Clustering auf Azure-VMs untersuchen, auf denen Windows Server 2022 ausgeführt wird.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

-   Bereitstellen von Azure-Computeressourcen, die zur Unterstützung von hochverfügbaren SAP NetWeaver-Bereitstellungen erforderlich sind

-   Konfigurieren des Betriebssystems von Azure-VMs unter Windows Server 2022 zur Unterstützung einer hochverfügbaren SAP NetWeaver-Bereitstellung

-   Bereitstellen von Azure-Netzwerkressourcen, die zur Unterstützung von hochverfügbaren SAP NetWeaver-Bereitstellungen erforderlich sind

## Anforderungen

-   Ein Microsoft Azure-Abonnement mit der ausreichenden Anzahl verfügbarer DSv2- und Dsv3-vCPUs (eine Standard_DS1_v2-VM mit jeweils einer vCPU und vier Standard_D4s_v3-VMs mit jeweils vier vCPUs) in der Azure-Region, die Sie für dieses Lab verwenden möchten

-   Ein Labcomputer mit Azure Cloud Shell-kompatiblem Webbrowser und Zugriff auf Azure

> **Hinweis:** Wählen Sie unbedingt eine Azure-Region für die Bereitstellung Ihrer Ressourcen aus, die Verfügbarkeitszonen unterstützt. Die Liste dieser Regionen finden Sie unter (https://docs.microsoft.com/en-us/azure/availability-zones/az-overview). Ziehen Sie **USA, Osten** oder **USA, Osten 2** in Betracht.

## Übung 1: Bereitstellen von Azure-Computeressourcen, die zur Unterstützung von hochverfügbaren SAP NetWeaver-Bereitstellungen erforderlich sind

Duration (Dauer): 50 Minuten

In dieser Übung stellen Sie Azure-Infrastrukturcomputekomponenten bereit, die zum Konfigurieren des Failoverclusterings auf Azure-VMs mit Windows Server 2022 erforderlich sind. Das umfasst die Bereitstellung eines Active Directory-Domänencontrollerpaars, gefolgt von einem Azure-VM-Paar mit Windows Server 2022. Jedes VM-Paar wird in separaten Verfügbarkeitszonen innerhalb desselben virtuellen Netzwerks bereitgestellt. Um die Bereitstellung von Domänencontrollern zu automatisieren, verwenden Sie eine Azure Resource Manager-Schnellstartvorlage, die über <https://aka.ms/az120-1bdeploy> verfügbar ist.

### Aufgabe 1: Bereitstellen eines Azure-VM-Paars mit hochverfügbaren Active Directory-Domänencontrollern mithilfe einer Bicep-Vorlage

1. Starten Sie auf dem Labcomputer einen Webbrowser, und navigieren Sie zum Azure-Portal unter **https://portal.azure.com**.

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
    $rgName = 'az12001b-ad-RG'
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
    $deploymentName = 'az1201b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
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

    - Navigieren Sie zum Blatt **az1201b-ad-RG**. Wählen Sie im vertikalen Navigationsmenü auf der linken Seite im Abschnitt **Einstellungen** die Option **Bereitstellungen** aus.

    - Wählen Sie auf dem Blatt **az1201b-ad-RG \| Bereitstellungen** den Bereitstellungsnamen aus, der mit dem Präfix **az1201b** beginnt, und wählen Sie auf dem Bereitstellungsblatt die Option **Erneut bereitstellen** aus.

    - Geben Sie auf dem Blatt **Benutzerdefinierte Bereitstellung** im Textfeld **Administratorkennwort** dasselbe Kennwort ein, das Sie während der ursprünglichen Bereitstellung verwendet haben, und wählen Sie **Überprüfen und erstellen** und dann **Erstellen** aus.

    - Warten Sie nicht, bis die erneute Bereitstellung abgeschlossen ist, sondern fahren Sie stattdessen mit der nächsten Aufgabe fort. Die erneute Bereitstellung sollte ungefähr drei Minuten dauern.

### Aufgabe 2: Bereitstellen eines Azure-VM-Paars mit Windows Server 2022 in verschiedenen Verfügbarkeitszonen

1. Navigieren Sie auf dem Lab-Computer im Azure-Portal zum Blatt **Virtuelle Computer**, klicken Sie auf **+ Erstellen**, und wählen Sie im Dropdownmenü **Azure-VM** aus.

1. Initiieren Sie auf dem Blatt **Virtuellen Computer erstellen** die Bereitstellung einer Azure-VM mit **Windows Server 2022 Datacenter: Azure Edition – Gen2** mit den folgenden Einstellungen (belassen Sie bei allen anderen die Standardwerte):
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Name der neuen Ressourcengruppe* **az12001b-cl-RG** |
    | **Name des virtuellen Computers** | **az12001b-cl-vm0** |
    | **Region** | *dieselbe Azure-Region, in der Sie die Azure-VMs in der vorherigen Aufgabe bereitgestellt haben* |
    | **Verfügbarkeitsoptionen** | **Verfügbarkeitszone** |
    | **Verfügbarkeitszone** | **Zone 1** |
    | **Image** | *Auswählen:* **Windows Server 2022 Datacenter: Azure Edition – Gen2** |
    | **Größe** | **Standard D4s v3** |
    | **Benutzername** | *denselben Benutzernamen, den Sie bei der Bereitstellung der Bicep-Vorlage weiter oben in dieser Übung angegeben haben* |
    | **Kennwort** | *dasselbe Kennwort, das Sie bei der Bereitstellung der Bicep-Vorlage weiter oben in dieser Übung angegeben haben* |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Möchten Sie eine vorhandene Windows Server-Lizenz verwenden?** | **Nein** |
    | **Typ des Betriebssystemdatenträgers** | **SSD Premium** |
    | **Virtuelles Netzwerk** | **adVNET** |
    | **Subnetzname** | *ein neues Subnetz namens* **clSubnet** |
    | **Subnetzadressbereich** | **10.0.1.0/24** |
    | **Öffentliche IP-Adresse** | *eine neue IP-Adresse namens* **az12001b-cl-vm0-ip** |
    | **NIC-Netzwerksicherheitsgruppe** | **Grundlegend**  |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Ein** |
    | **Optionen für den Lastenausgleich** | **Keine** |
    | **Aktivieren einer systemseitig zugewiesenen verwalteten Identität** | **Deaktiviert** |
    | **Mit Azure AD anmelden** | **Deaktiviert** |
    | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
    | **Option für die Patchorchestrierung** | **Manuelle Updates** |
    | **Startdiagnose** | **Deaktivieren** |
    | **Erweiterungen** | *Keine* |
    | **Tags** | *Keine* |

1. Warten Sie nicht, bis die Bereitstellung abgeschlossen ist, sondern fahren Sie mit dem nächsten Schritt fort.

1. Stellen Sie eine weitere Azure-VM mit **Windows Server 2022 Datacenter: Azure Edition – Gen2** mit den folgenden Einstellungen bereit:
     
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Namen der Ressourcengruppe, die Sie beim Bereitstellen der ersten Azure-VM mit **Windows Server 2022 Datacenter: Azure Edition – Gen2** in dieser Aufgabe verwendet haben* |
    | **Name des virtuellen Computers** | **az12001b-cl-vm1** |
    | **Region** | *dieselbe Azure-Region, in der Sie die erste Azure-VM mit **Windows Server 2022 Datacenter: Azure Edition – Gen2** in dieser Aufgabe verwendet haben* |
    | **Verfügbarkeitsoptionen** | **Verfügbarkeitszone** |
    | **Verfügbarkeitszone** | **Zone 2** |
    | **Image** | *Auswählen:* **Windows Server 2022 Datacenter: Azure Edition – Gen2** |
    | **Größe** | **Standard D4s v3** |
    | **Benutzername** | *denselben Benutzernamen, den Sie bei der Bereitstellung der Bicep-Vorlage weiter oben in dieser Übung angegeben haben* |
    | **Kennwort** | *dasselbe Kennwort, das Sie bei der Bereitstellung der Bicep-Vorlage weiter oben in dieser Übung angegeben haben* |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Möchten Sie eine vorhandene Windows Server-Lizenz verwenden?** | **Nein** |
    | **Typ des Betriebssystemdatenträgers** | **SSD Premium** |
    | **Virtuelles Netzwerk** | **adVNET** |
    | **Subnetzname** | **clSubnet** |
    | **Öffentliche IP-Adresse** | *eine neue IP-Adresse namens* **az12001b-cl-vm1-ip** |
    | **NIC-Netzwerksicherheitsgruppe** | **Grundlegend**  |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Ein** |
    | **Optionen für den Lastenausgleich** | **Keine** |
    | **Mit Azure AD anmelden** | **Deaktiviert** |
    | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
    | **Option für die Patchorchestrierung** | **Manuelle Updates** |
    | **Startdiagnose** | **Deaktivieren** |
    | **Erweiterungen** | *Keine* |
    | **Tags** | *Keine* |

1. Warten Sie, bis die Bereitstellung abgeschlossen wurde. Dies sollte einige Minuten dauern.

### Aufgabe 3: Erstellen und Konfigurieren von Azure-VM-Datenträgern

1. Starten Sie im Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die die Ressourcen enthält, die Sie in der vorherigen Aufgabe bereitgestellt haben:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die erste Gruppe von vier verwalteten Datenträgern zu erstellen, die Sie an die erste Azure-VM anfügen, die Sie in der vorherigen Aufgabe bereitgestellt haben:

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location
    
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm0').Zones

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die zweite Gruppe von vier verwalteten Datenträgern zu erstellen, die Sie an die zweite Azure-VM anfügen, die Sie in der vorherigen Aufgabe bereitgestellt haben:

    ```
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm1').Zones
    
    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone
        
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1. Navigieren Sie im Azure-Portal zum Blatt der ersten Azure-VM, die Sie in der vorherigen Aufgabe bereitgestellt haben (**az12001b-cl-vm0**).

1. Navigieren Sie vom Blatt **az12001b-cl-vm0** zum Blatt **az12001b-cl-vm0 – Datenträger**.

1. Fügen Sie über das Blatt **az12001b-cl-vm0 – Datenträger** mehrere Datenträger mit den folgenden Einstellungen an „az12001b-cl-vm0“ an:
   
   | Einstellung | Wert |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Name des Datenträgers** | **az12001b-cl-vm0-DataDisk0** |
   | **Ressourcengruppe** | *der Namen der Ressourcengruppe, die Sie beim Bereitstellen des Azure-VM-Paars mit **Windows Server 2022 Datacenter** in der vorherigen Aufgabe verwendet haben* |
   | **HOSTZWISCHENSPEICHERUNG** | **Schreibgeschützt** |

1. Wiederholen Sie den vorherigen Schritt, um die verbleibenden drei Datenträger mit dem Präfix **az12001b-cl-vm0-DataDisk** anzufügen, damit es insgesamt vier sind. Weisen Sie die LUN-Nummer zu, die dem letzten Zeichen des Datenträgernamens entspricht. Legen Sie für den letzten Datenträger (LUN **3**) die Option „HOSTZWISCHENSPEICHERUNG“ auf **Keine** fest.

1. Speichern Sie die Änderungen. 

1. Navigieren Sie im Azure-Portal zum Blatt der zweiten Azure-VM (**az12001b-cl-vm1**), die Sie in der vorherigen Aufgabe bereitgestellt haben.

1. Navigieren Sie vom Blatt **az12001b-cl-vm1** zum Blatt **az12001b-cl-vm1 – Datenträger**.

1. Fügen Sie über das Blatt **az12001b-cl-vm1 – Datenträger** mehrere Datenträger mit den folgenden Einstellungen an „az12001b-cl-vm1“ an:
     
   | Einstellung | Wert |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Name des Datenträgers** | **az12001b-cl-vm1-DataDisk0** |
   | **Ressourcengruppe** | *der Namen der Ressourcengruppe, die Sie beim Bereitstellen des Azure-VM-Paars mit **Windows Server 2022 Datacenter** in der vorherigen Aufgabe verwendet haben* |
   | **HOSTZWISCHENSPEICHERUNG** | **Schreibgeschützt** |

1. Wiederholen Sie den vorherigen Schritt, um die verbleibenden drei Datenträger mit dem Präfix **az12001b-cl-vm1-DataDisk** anzufügen, damit es insgesamt vier sind. Weisen Sie die LUN-Nummer zu, die dem letzten Zeichen des Datenträgernamens entspricht. Legen Sie für den letzten Datenträger (LUN **3**) die Option „HOSTZWISCHENSPEICHERUNG“ auf **Keine** fest.

1. Speichern Sie die Änderungen. 

> **Result**: Durch den Abschluss dieser Übung haben Sie die Azure-Computeressourcen bereitgestellt, die erforderlich sind, um hochverfügbare SAP NetWeaver-Bereitstellungen zu unterstützen.


## Übung 2: Konfigurieren des Betriebssystems von Azure-VMs mit Windows Server 2022 Datacenter für die Unterstützung einer hochverfügbaren SAP NetWeaver-Installation

Duration (Dauer): 40 Minuten

### Aufgabe 1: Einbinden der Windows Server 2022 Datacenter-VMs in die Active Directory-Domäne

   > **Hinweis:** Bevor Sie diese Aufgabe starten, muss die Vorlagenbereitstellung, die Sie in der letzten Aufgabe der vorherigen Übung initiiert haben, erfolgreich abgeschlossen sein. 

1. Navigieren Sie im Azure-Portal zum Blatt des virtuellen Netzwerks **adVNET**, das in der ersten Übung dieses Labs automatisch bereitgestellt wurde.

1. Öffnen Sie das Blatt **adVNET – DNS-Server**. Sie sehen, dass das virtuelle Netzwerk mit den privaten IP-Adressen konfiguriert ist, die den Domänencontrollern, die in der ersten Übung dieses Labs bereitgestellt wurden, als DNS-Server zugewiesen sind.

1. Starten Sie im Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die das Azure-VM-Paar mit **Windows Server 2022 Datacenter: Azure Edition – Gen2** enthält, das Sie in der vorherigen Übung bereitgestellt haben:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Azure-VMs mit Windows Server 2022 einzubinden, die Sie in der zweiten Aufgabe der vorherigen Übung in der Active Directory-Domäne **adatum.com** bereitgestellt haben. Ersetzen Sie die Platzhalter `<username>` und `<password>` durch den Namen und das Kennwort des Administratorbenutzerkontos, das Sie beim Bereitstellen der Bicep-Vorlage in der ersten Übung dieser Übung angegeben haben:

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\<username>", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1. Warten Sie, bis das Skript abgeschlossen ist, bevor Sie mit der nächsten Aufgabe fortfahren.


### Aufgabe 2: Konfigurieren des Speichers auf Azure-VMs mit Windows Server 2022 zur Unterstützung einer hochverfügbaren SAP NetWeaver-Installation

1. Navigieren Sie im Azure-Portal zum Blatt der VM **az12001b-cl-vm0**, die Sie in der ersten Übung dieses Labs bereitgestellt haben.

1. Stellen Sie über das Blatt **az12001b-cl-vm0** eine Verbindung mit dem VM-Gastbetriebssystem über einen Remotedesktop her. Wenn Sie zur Authentifizierung aufgefordert werden, geben Sie die Anmeldeinformationen des Administratorbenutzerkontos an, das Sie beim Bereitstellen der Bicep-Vorlage in der ersten Übung dieses Labs angegeben haben. 

    > **Hinweis:** Stellen Sie sicher, dass Sie sich mit dem **ADATUM**-Domänenkonto und nicht mit dem Konto auf Betriebssystemebene anmelden (d. h., dass dem Benutzernamen das Präfix **ADATUM\\** vorangestellt ist).

1. Navigieren Sie in der RDP-Sitzung zu „az12001b-cl-vm0“, und dann im Server-Manager zum Knoten **Datei- und Speicherdienste** -> **Server**. 

1. Navigieren Sie zur Ansicht **Speicherpools**, und überprüfen Sie, ob alle Datenträger angezeigt werden, die Sie in der vorherigen Übung an die Azure-VM angefügt haben.

1. Verwenden Sie den **Assistenten zum Erstellen eines neuen Speicherpools**, um einen neuen Speicherpool mit den folgenden Einstellungen zu erstellen:
     
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **Datenspeicherpool** |
    | **Physische Datenträger** | *Wählen Sie die drei Datenträger mit Datenträgernummern aus, die den ersten drei LUN-Nummern (0 bis 2) entsprechen, und legen Sie die Zuweisung auf* **Automatisch** fest. |

    > **Hinweis:** Verwenden Sie den Eintrag in der Spalte **Chassis**, um die **LUN**-Nummer zu ermitteln.

1. Verwenden Sie den **Assistenten für neue virtuelle Datenträger**, um einen neuen virtuellen Datenträger mit den folgenden Einstellungen zu erstellen:
     
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name des virtuellen Datenträgers** | **Virtueller Datenträger** |
    | **Speicherlayout** | **Einfach** |
    | **Bereitstellung** | **Korrigiert** |
    | **Größe** | **Maximale Größe** |

1. Verwenden Sie den **Assistenten für neue Volumes**, um ein neues Volume mit den folgenden Einstellungen zu erstellen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Server und Datenträger** | *Standardwerte akzeptieren* |
    | **Größe** | *Standardwerte akzeptieren* |
    | **Laufwerkbuchstabe** | **M** |
    | **Dateisystem** | **ReFS** |
    | **Größe der Zuordnungseinheit** | **Standard** |
    | **Volumebezeichnung** | **Daten** |

1. Verwenden Sie in der Ansicht **Speicherpools** den **Assistenten zum Erstellen eines neuen Speicherpools**, um einen neuen Speicherpool mit den folgenden Einstellungen zu erstellen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **Protokollspeicherpool** |
    | **Physische Datenträger** | *Wählen Sie den letzten der vier Datenträger aus, und legen Sie die Zuweisung auf* **Automatisch** fest. |

1. Verwenden Sie den **Assistenten für neue virtuelle Datenträger**, um einen neuen virtuellen Datenträger mit den folgenden Einstellungen zu erstellen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name des virtuellen Datenträgers** | **Protokoll für virtuellen Datenträger** |
    | **Speicherlayout** | **Einfach** |
    | **Bereitstellung** | **Korrigiert** |
    | **Größe** | **Maximale Größe** |

1. Verwenden Sie den **Assistenten für neue Volumes**, um ein neues Volume mit den folgenden Einstellungen zu erstellen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Server und Datenträger** | *Standardwerte akzeptieren* |
    | **Größe** | *Standardwerte akzeptieren* |
    | **Laufwerkbuchstabe** | **L** |
    | **Dateisystem** | **ReFS** |
    | **Größe der Zuordnungseinheit** | **Standard** |
    | **Volumebezeichnung** | **Protokoll** |

1. Wiederholen Sie den vorherigen Schritt in dieser Aufgabe, um den Speicher auf „az12001b-cl-vm1“ zu konfigurieren.

### Aufgabe 3: Vorbereiten der Konfiguration des Failoverclusterings auf Azure-VMs mit Windows Server 2022 zur Unterstützung einer hochverfügbaren SAP NetWeaver-Installation

1. Starten Sie in der RDP-Sitzung auf „az12001b-cl-vm0“ eine Windows PowerShell ISE-Sitzung, und installieren Sie die Features für Failoverclustering und Remoteverwaltungstools sowohl auf „az12001b-cl-vm0“ als auch auf „az12001b-cl-vm1“, indem Sie Folgendes ausführen:

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Hinweis:** Dieser Vorgang führt zu einem Neustart des Gastbetriebssystems beider Azure-VMs.

1. Klicken Sie auf dem Labcomputer im Azure-Portal auf **+ Ressource erstellen**.

1. Initiieren Sie auf dem Blatt **Neu** die Erstellung eines neuen **Speicherkontos** mit den folgenden Einstellungen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements* |
    | **Ressourcengruppe** | *der Name der Ressourcengruppe, die das Azure-VM-Paar mit **Windows Server 2022 Datacenter** enthält, die Sie in der vorherigen Übung bereitgestellt haben* |
    | **Speicherkontoname** | *ein eindeutiger Name aus 3 bis 24 Buchstaben und Ziffern* |
    | **Location** | *dieselbe Azure-Region, in der Sie die Azure-VMs in der vorherigen Übung bereitgestellt haben* |
    | **Leistung** | **Standard** |
    | **Redundanz** | **Lokal redundanter Speicher (LRS)** |
    | **Konnektivitätsmethode** | **Öffentlicher Endpunkt (alle Netzwerke)** |
    | **Sichere Übertragung für REST-API-Vorgänge erforderlich** | **Enabled** |
    | **Große Dateifreigaben** | **Disabled** |
    | **Vorläufiges Löschen für Blobs, Container und Dateien** | **Disabled** |
    | **Hierarchischer Namespace** | **Disabled** |

### Aufgabe 4: Konfigurieren des Failoverclusterings auf Azure-VMs mit Windows Server 2022 zur Unterstützung einer hochverfügbaren SAP NetWeaver-Installation

1. Navigieren Sie im Azure-Portal zum Blatt der VM **az12001b-cl-vm0**, die Sie in der ersten Übung dieses Labs bereitgestellt haben.

1. Stellen Sie über das Blatt **az12001b-cl-vm0** eine Verbindung mit dem VM-Gastbetriebssystem über einen Remotedesktop her. Wenn Sie zur Authentifizierung aufgefordert werden, geben Sie die Anmeldeinformationen des Administratorbenutzerkontos an, das Sie beim Bereitstellen der Bicep-Vorlage in der ersten Übung dieses Labs angegeben haben.

1. Starten Sie in der RDP-Sitzung für „az12001b-cl-vm0“ in Server-Manager über das Menü **Extras** das **Active Directory-Verwaltungscenter**.

1. Erstellen Sie im Active Directory-Verwaltungscenter eine neue Organisationseinheit namens **Cluster** im Stammverzeichnis der Domäne „adatum.com“.

1. Verschieben Sie im Active Directory-Verwaltungscenter die Computerkonten von **az12001b-cl-vm0** und **az12001b-cl-vm1** aus dem Container **Computer** in die Organisationseinheit **Cluster**.

1. Starten Sie in der RDP-Sitzung für „az12001b-cl-vm0“ eine Windows PowerShell ISE-Sitzung, und erstellen Sie einen neuen Cluster, indem Sie Folgendes ausführen:

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1. Wechseln Sie innerhalb der RDP-Sitzung für „az12001b-cl-vm0“ zur Konsole **Active Directory-Verwaltungscenter**.

1. Navigieren Sie im Active Directory-Verwaltungscenter zur Organisationseinheit **Cluster**, und öffnen Sie deren **Eigenschaftenfenster**. 

1. Navigieren Sie im Fenster **Eigenschaften** der Organisationseinheit **Cluster** zum Abschnitt **Erweiterungen**, und öffnen Sie die Registerkarte **Sicherheit**. 

1. Klicken Sie auf der Registerkarte **Sicherheit** auf die Schaltfläche **Erweitert**, um das Fenster **Erweiterte Sicherheitseinstellungen für Cluster** zu öffnen. 

1. Klicken Sie auf der Registerkarte **Berechtigungen** des Fensters **Erweiterte Sicherheitseinstellungen für Cluster** auf **Hinzufügen**.

1. Klicken Sie im Fenster **Berechtigungseintrag für Cluster** auf **Prinzipal auswählen**.

1. Klicken Sie im Dialogfeld **Benutzer, Dienstkonto oder Gruppe auswählen** auf **Objekttypen**, aktivieren Sie das Kontrollkästchen neben dem Eintrag **Computer**, und klicken Sie auf **OK**. 

1. Geben Sie im Dialogfeld **Benutzer, Computer, Dienstkonto oder Gruppe auswählen** im Dialogfeld **Namen des auszuwählenden Objekts eingeben** den Wert **az12001b-cl-cl0** ein, und klicken Sie auf **OK**.

1. Vergewissern Sie sich im Fenster **Berechtigungseintrag für Cluster**, dass **Zulassen** in der Dropdownliste **Typ** angezeigt wird. Wählen Sie dann in der Dropdownliste **Gilt für** die Option **Dieses und alle untergeordneten Objekte** aus. Aktivieren Sie in der Liste **Berechtigungen** die Kontrollkästchen **Computerobjekte erstellen** und **Computerobjekte löschen**, und klicken Sie zweimal auf **OK**.

1. Installieren Sie in der Windows PowerShell ISE-Sitzung das PowerShell-Modul „Az“, indem Sie Folgendes ausführen:

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Authentifizieren Sie sich bei der Windows PowerShell ISE-Sitzung mithilfe Ihrer Azure AD-Anmeldeinformationen, indem Sie Folgendes ausführen:

    ```
    Add-AzAccount
    ```

    > **Hinweis:** Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Geschäfts-, Schul- oder Uni- oder persönlichen Microsoft-Konto mit der Besitzer- oder der Mitwirkendenrolle beim Azure-Abonnement an, das Sie für dieses Lab verwenden.

1. Führen Sie in der Windows PowerShell ISE-Sitzung Folgendes aus, um das Cloudzeugenquorum des neuen Clusters festzulegen:

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Zur Überprüfung der resultierenden Konfiguration starten Sie in der RDP-Sitzung für „az12001b-cl-vm0“ über das Menü **Extras** in Server-Manager den **Failovercluster-Manager**.

1. Überprüfen Sie in der Konsole des **Failovercluster-Managers** die Clusterkonfiguration **az12001b-cl-cl0**, einschließlich der Knoten sowie der Zeugen- und Netzwerkeinstellungen. Sie sehen, dass der Cluster keinen freigegebenen Speicher hat.

1. Beenden Sie die RDP-Sitzung für „az12001b-cl-vm0“.

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie das Betriebssystem von Azure-VMs mit Windows Server 2022 konfiguriert, um eine hochverfügbare SAP NetWeaver-Installation zu unterstützen.


## Übung 3: Bereitstellen von Azure-Netzwerkressourcen, die zur Unterstützung von hochverfügbaren SAP NetWeaver-Bereitstellungen erforderlich sind

Dauer: 30 Minuten

In dieser Übung implementieren Sie Azure Load Balancer-Instanzen, um gruppierte Installationen von SAP NetWeaver aufzunehmen.

### Aufgabe 1: Konfigurieren von Azure-VMs, um das Einrichten des Lastenausgleichs zu erleichtern

   > **Hinweis:** Da Sie ein Azure Load Balancer-Paar der Standard-SKU einrichten, müssen Sie zuerst die öffentlichen IP-Adressen entfernen, die Netzwerkadaptern der beiden Azure-VMs zugeordnet sind, die als Back-End-Pool für den Lastenausgleich dienen.

1. Navigieren Sie auf dem Laborcomputer im Azure-Portal zum Blatt der Azure-VM **az12001b-cl-vm0**. 

1. Navigieren Sie vom Blatt **az12001b-cl-vm0** zum Blatt der öffentlichen IP-Adresse **az12001b-cl-vm0-ip**, die dem Netzwerkadapter zugeordnet ist.

1. Heben Sie über das Blatt **az12001b-cl-vm0-ip** zuerst die Zuordnung der öffentlichen IP-Adresse zur Netzwerkschnittstelle auf, und löschen Sie sie dann.

1. Navigieren Sie im Azure-Portal zum Blatt der Azure-VM **az12001b-cl-vm1**. 

1. Navigieren Sie vom Blatt **az12001b-cl-vm1** zum Blatt der öffentlichen IP-Adresse **az12001b-cl-vm1-ip**, die dem Netzwerkadapter zugeordnet.

1. Heben Sie über das Blatt **az12001b-cl-vm1-ip** zuerst die Zuordnung der öffentlichen IP-Adresse zur Netzwerkschnittstelle auf, und löschen Sie sie dann.

1. Navigieren Sie im Azure-Portal zum Blatt der Azure-VM **az12001a-vm0**.

1. Navigieren Sie vom Blatt **az12001a-vm0** zum Blatt **Netzwerk**. 

1. Navigieren Sie vom Blatt **az12001a-vm0 – Netzwerk** zur Netzwerkschnittstelle von „az12001a-vm0“. 

1. Navigieren Sie vom Blatt der Netzwerkschnittstelle von „az12001a-vm0“ zum Blatt „IP-Konfigurationen“, und öffnen Sie dort das Blatt **ipconfig1**.

1. Legen Sie auf dem Blatt **ipconfig1** die private IP-Adresszuweisung auf **Statisch** fest, und speichern Sie die Änderung.

1. Navigieren Sie im Azure-Portal zum Blatt der Azure-VM **az12001a-vm1**.

1. Navigieren Sie vom Blatt **az12001a-vm1** zum Blatt **Netzwerk**. 

1. Navigieren Sie vom Blatt **az12001a-vm1 – Netzwerk** zur Netzwerkschnittstelle von „az12001a-vm1“. 

1. Navigieren Sie vom Blatt der Netzwerkschnittstelle von „az12001a-vm1“ zum Blatt „IP-Konfigurationen“, und öffnen Sie dort das Blatt **ipconfig1**.

1. Legen Sie auf dem Blatt **ipconfig1** die private IP-Adresszuweisung auf **Statisch** fest, und speichern Sie die Änderung.

### Aufgabe 2: Erstellen und Konfigurieren von Azure Load Balancer-Instanzen für eingehenden Datenverkehr

1. Klicken Sie im Azure-Portal auf **+ Ressource erstellen**.

1. Initiieren Sie über das Blatt **Neu** die Erstellung einer neuen Azure Load Balancer-Instanz mit den folgenden Einstellungen:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements* |
    | **Ressourcengruppe** | *der Name der Ressourcengruppe, die das Azure-VM-Paar mit **Windows Server 2022 Datacenter** enthält, die Sie in der ersten Übung dieses Labs bereitgestellt haben* |
    | **Name** | **az12001b-cl-lb0** |
    | **Region** | dieselbe Azure-Region, in der Sie Azure-VMs in der ersten Übung dieses Labs bereitgestellt haben* |
    | **SKU** | **Standard** |
    | **Typ** | **Intern** |
    | **Name der Front-End-IP-Adresse** | **frontend-ip1** |
    | **Virtuelles Netzwerk** | **adVNET** |
    | **Subnetz** | **clSubnet** |
    | **IP-Adresszuweisung** | **Statisch** |
    | **IP-Adresse** | **10.0.1.240** |
    | **Verfügbarkeitszone** | **Zonenredundant** |

1. Warten Sie, bis der Lastenausgleich bereitgestellt wurde, und navigieren Sie dann im Azure-Portal zum zugehörigen Blatt.

1. Fügen Sie über das Blatt **az12001b-cl-lb0** einen Back-End-Pool mit den folgenden Einstellungen hinzu:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **az12001b-cl-lb0-bepool** |
    | **Virtuelles Netzwerk** | **adVNET** |
    | **Konfiguration des Back-End-Pools** | **IP-Adresse** |
    | **IP-Adresse** | **10.0.1.4** Ressourcenname **az1201b-cl-vm0** |
    | **IP-Adresse** | **10.0.1.5** Ressourcenname **az1201b-cl-vm1** |

1. Fügen Sie über das Blatt **az12001b-cl-lb0** einen Integritätstest mit den folgenden Einstellungen hinzu:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **az12001b-cl-lb0-hprobe** |
    | **Protokoll** | **TCP** |
    | **Port** | **59999** |
    | **Intervall** | **5** *Sekunden* |
    | **Fehlerhafter Schwellenwert** | **2** *aufeinanderfolgende Fehler* |

1. Fügen Sie über das Blatt **az12001b-cl-lb0** eine Netzwerklastenausgleichsregel mit den folgenden Einstellungen hinzu:
     
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **az12001b-cl-lb0-lbruletcp1433** |
    | **IP-Version** | **IPv4** |
    | **Frontend IP address** (Front-End-IP-Adresse) | **10.0.1.240 (LoadBalancerFrontEnd)** |
    | **Hochverfügbarkeitsports** | **Disabled** |
    | **Protokoll** | **TCP** |
    | **Port** | **1433** |
    | **Back-End-Port** | **1433** |
    | **Back-End-Pool** | **az12001b-cl-lb0-bepool (2 VMs)** |
    | **Integritätstest** | **az12001b-cl-lb0-hprobe (TCP:59999)** |
    | **Sitzungspersistenz** | **None** |
    | **Leerlaufzeitüberschreitung (Minuten)** | **4** |
    | **TCP-Zurücksetzung** | **Disabled** |
    | **Floating IP (Direct Server Return)** | **Enabled** |

### Aufgabe 3: Erstellen und Konfigurieren von Azure Load Balancer-Instanzen für ausgehenden Datenverkehr

1. Starten Sie über das Azure-Portal in Cloud Shell eine PowerShell-Sitzung. 

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die das Azure-VM-Paar mit **Windows Server 2022 Datacenter** enthält, das Sie in der ersten Übung dieses Labs bereitgestellt haben:

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die öffentliche IP-Adresse zu erstellen, die vom zweiten Lastenausgleich verwendet werden soll:

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den zweiten Lastenausgleich zu erstellen:

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1. Schließen Sie den Cloud Shell-Bereich.

1. Navigieren Sie im Azure-Portal zum Blatt, auf dem die Eigenschaften der Azure Load Balancer-Instanz **az12001b-cl-lb1** angezeigt werden.

1. Klicken Sie auf dem Blatt **az12001b-cl-lb1** auf **Back-End-Pools**.

1. Klicken Sie auf dem Blatt **az12001b-cl-lb1 – Back-End-Pools** auf **az12001b-cl-lb1-bepool**.

1. Geben Sie auf dem Blatt **az12001b-cl-lb1-bepool** die folgenden Einstellungen an, und klicken Sie auf **Speichern**:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Virtuelles Netzwerk** | **adVNET (4 VMs)** |
    | **Virtueller Computer** | **az12001b-cl-vm0**  IP-ADRESSE: **ipconfig1** |
    | **Virtueller Computer** | **az12001b-cl-vm1**  IP-ADRESSE: **ipconfig1** |

1. Klicken Sie auf dem Blatt **az12001b-cl-lb1** auf **Integritätstests**.

1. Fügen Sie über das Blatt **az12001b-cl-lb1 – Integritätstests** einen Integritätstest mit den folgenden Einstellungen hinzu:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **az12001b-cl-lb1-hprobe** |
    | **Protokoll** | **TCP** |
    | **Port** | **80** |
    | **Intervall** | **5** *Sekunden* |
    | **Fehlerhafter Schwellenwert** | **2** *aufeinanderfolgende Fehler* |

1. Klicken Sie auf dem Blatt **az12001b-cl-lb1** auf **Lastenausgleichsregeln**.

1. Fügen Sie über das Blatt **az12001b-cl-lb1 – Lastenausgleichsregeln** eine Netzwerklastenausgleichsregel mit den folgenden Einstellungen hinzu:
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Name** | **az12001b-cl-lb1-lbharule** |
    | **IP-Version** | **IPv4** |
    | **Frontend IP address** (Front-End-IP-Adresse) | *Akzeptieren Sie den Standardwert.* |
    | **Hochverfügbarkeitsports** | **Disabled** |
    | **Protokoll** | **TCP** |
    | **Port** | **80** |
    | **Back-End-Port** | **80** |
    | **Back-End-Pool** | **az12001b-cl-lb1-bepool (2 VMs)** |
    | **Integritätstest** | **az12001b-cl-lb1-hprobe (TCP:80)** |
    | **Sitzungspersistenz** | **None** |
    | **Leerlaufzeitüberschreitung (Minuten)** | **4** |
    | **TCP-Zurücksetzung** | **Disabled** |
    | **Floating IP (Direct Server Return)** | **Disabled** |

### Aufgabe 4: Bereitstellen eines Bastion Hosts

   > **Hinweis:** Da zwei gruppierte Azure-VMs nicht mehr direkt über das Internet zugänglich sind, stellen Sie eine Azure-VM mit Windows Server 2022 Datacenter bereit, die als Bastion Host dient. 

1. Navigieren Sie auf dem Lab-Computer im Azure-Portal zum Blatt **Virtuelle Computer**, klicken Sie auf **+ Erstellen**, und wählen Sie im Dropdownmenü **Azure-VM** aus.

1. Initiieren Sie auf dem Blatt **Virtuellen Computer erstellen** die Bereitstellung einer Azure-VM mit **Windows Server 2022 Datacenter: Azure Edition – Gen2** mit den folgenden Einstellungen bereit:
     
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *der Namen der Ressourcengruppe, die das Azure-VM-Paar mit **Windows Server 2022 Datacenter Azure Edition – Gen2** enthält, das Sie in der ersten Übung dieses Labs bereitgestellt haben* |
    | **Name des virtuellen Computers** | **az12001b-vm2** |
    | **Region** | *dieselbe Azure-Region, in der Sie Azure-VMs in der ersten Übung dieses Labs bereitgestellt haben* |
    | **Verfügbarkeitsoptionen** | **Keine Infrastrukturredundanz erforderlich** |
    | **Image** | *Auswählen:* **Windows Server 2022 Datacenter: Azure Edition – Gen2** |
    | **Größe** | **Standard DS1 v2*** oder ähnlich* |
    | **Benutzername** | *derselbe Benutzernamen, den Sie bei der Bereitstellung der Bicep-Vorlage in der ersten Übung dieses Labs angegeben haben* |
    | **Kennwort** | *dasselbe Kennwort, das Sie bei der Bereitstellung der Bicep-Vorlage in der ersten Übung dieses Labs angegeben haben* |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Möchten Sie eine vorhandene Windows Server-Lizenz verwenden?** | **Nein** |
    | **Typ des Betriebssystemdatenträgers** | **HDD Standard** |
    | **Virtuelles Netzwerk** | **adVNET** |
    | **Subnetzname** | *ein neues Subnetz namens* **BastionSubnet** |
    | **Subnetzadressbereich** | **10.0.255.0/24** |
    | **Öffentliche IP-Adresse** | *eine neue IP-Adresse namens* **az12001b-vm2-ip** |
    | **NIC-Netzwerksicherheitsgruppe** | **Grundlegend**  |
    | **Öffentliche Eingangsports** | **Ausgewählte Ports zulassen** |
    | **Ausgewählte eingehende Ports** | **RDP (3389)** |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Deaktiviert** |
    | **Optionen für den Lastenausgleich** | **Keine** |
    | **Aktivieren einer systemseitig zugewiesenen verwalteten Identität** | **Deaktiviert** |
    | **Mit Azure AD anmelden** | **Deaktiviert** |
    | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
    | **Startdiagnose** | **Deaktivieren** |
    | **Diagnose des Gastbetriebssystems aktivieren** | **Deaktiviert** |
    | **Erweiterungen** | *Keine* |
    | **Tags** | *Keine* |

1. Warten Sie, bis die Bereitstellung abgeschlossen wurde. Dies sollte einige Minuten dauern.

1. Stellen Sie über RDP eine Verbindung mit der neu bereitgestellten Azure-VM her. 

1. Stellen Sie innerhalb der RDP-Sitzung für „az12001b-vm2“ sicher, dass Sie eine RDP-Sitzung sowohl für „az12001b-cl-vm0“ als auch für „az12001b-cl-vm1“ über deren private IP-Adressen (10.0.1.4 bzw. 10.0.1.5) einrichten können. 

> **Result**: Durch den Abschluss dieser Übung haben Sie die Azure-Netzwerkressourcen bereitgestellt, die erforderlich sind, um hochverfügbare SAP NetWeaver-Bereitstellungen zu unterstützen.

## Übung 4: Entfernen der Labressourcen

Dauer: 10 Minuten

In dieser Übung entfernen Sie die in diesem Lab bereitgestellten Ressourcen.

#### Aufgabe 1: Cloud Shell öffnen

1. Klicken Sie oben im Portal auf das **Cloud Shell**-Symbol, um den Cloud Shell-Bereich zu öffnen, und wählen Sie PowerShell als Shell aus.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `$resourceGroupName` auf den Namen der Ressourcengruppe festzulegen, die das Azure-VM-Paar mit **Windows Server 2022 Datacenter** enthält, das Sie in der ersten Übung dieses Labs bereitgestellt haben:

    ```
    $resourceGroupNamePrefix = 'az12001b-'
    ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um alle Ressourcengruppen aufzulisten, die Sie in diesem Lab erstellt haben:

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
