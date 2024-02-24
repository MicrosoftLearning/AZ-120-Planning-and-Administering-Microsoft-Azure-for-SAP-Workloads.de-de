---
lab:
  title: 02a – Implementieren von Linux-Clustering auf Azure-VMs
  module: Module 02 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 Modul 2: Erkunden der Grundlagen von IaaS für SAP in Azure
# Lab 2a: Implementieren von Linux-Clustering auf Azure-VMs

Geschätzte Dauer: 90 Minuten

Alle Aufgaben in diesem Lab werden über das Azure-Portal ausgeführt (einschließlich der Bash Cloud Shell-Sitzung)  

   > **Hinweis:** Wenn Sie keine Cloud Shell verwenden, muss auf dem virtuellen Labcomputer die Azure CLI installiert sein [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows) und ein SSH-Client wie PuTTY enthalten [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) sein.

Labdateien: keine

## Szenario
  
Als Vorbereitung für die SAP HANA-Bereitstellung in Azure möchte die Adatum Corporation den Prozess der Implementierung von Clustering auf Azure-VMs untersuchen, auf denen die SUSE-Distribution von Linux ausgeführt wird.

## Ziele
  
In diesem Lab lernen Sie Folgendes:

- Bereitstellen von Azure-Computeressourcen, die zur Unterstützung von hochverfügbaren SAP HANA-Bereitstellungen erforderlich sind

- Konfigurieren des Betriebssystems von Azure-VMs unter Linux zur Unterstützung einer hochverfügbaren SAP HANA Installation

- Bereitstellen von Azure-Netzwerkressourcen, die zur Unterstützung von hochverfügbaren SAP HANA-Bereitstellungen erforderlich sind

## Anforderungen

- Ein Microsoft Azure-Abonnement mit der ausreichenden Anzahl verfügbarer DSv3-vCPUs (2 x 4) und DSv2-vCPUs (1 x 1)

- Ein Labcomputer mit Azure Cloud Shell-kompatiblem Webbrowser und Zugriff auf Azure

## Übung 1: Bereitstellen von Azure-Computeressourcen, die zur Unterstützung von hochverfügbaren SAP HANA-Bereitstellungen erforderlich sind

Dauer: 30 Minuten

In dieser Übung stellen Sie Azure-Infrastruktur-Computerkomponenten bereit, die zum Konfigurieren von Linux-Clustering erforderlich sind. Dies umfasst das Erstellen eines Paars von Azure-VMs, auf denen Linux SUSE ausgeführt wird, in der gleichen Verfügbarkeitsgruppe und Bereitstellung von Azure Bastion.

### Aufgabe 1: Bereitstellen von Azure-VMs mit Linux SUSE

1. Starten Sie auf dem Laborcomputer einen Webbrowser und navigieren Sie zum Azure-Portal unter **https://portal.azure.com**.

1. Wenn Sie dazu aufgefordert werden, melden Sie sich mit dem Geschäfts-, Schul- oder Uni- oder persönlichen Microsoft-Konto mit der Besitzer- oder der Mitwirkendenrolle beim Azure-Abonnement an, das Sie für dieses Labor verwenden.

1. Verwenden Sie im Azure-Portal oben auf der Azure-Portalseite das **Textfeld "Ressourcen, Dienste und Dokumente** suchen" und navigieren Sie zum Blatt " **Näherungplatzierungsgruppen"**, und wählen Sie auf dem **Blatt "Näherungsplatzierungsgruppen"** die Option **"+erstellen**" aus.

1. Geben Sie auf der **Registerkarte "Grundlagen"** des Blatts **"Näherungsplatzierungsgruppen** erstellen" die folgenden Einstellungen an, und wählen Sie " **Überprüfen+ erstellen**" aus:

    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *Der Name Ihres Azure-Abonnements*  |
    | Abschnitt **Ressourcengruppe** | Wählen Sie **Neu erstellen** aus, geben Sie **az12001a-RG** ein und wählen Sie dann **OK** |
    | **Region** | *Die Azure-Region, in der Sie über ausreichende vCPU-Kontingente verfügen* |
    | **Name der Näherungsplatzierungsgruppe** | Wählen Sie **az12001a-ppg** aus |
    | **Absicht-Details** | **Standard D4s v3** |

   > **Hinweis:** Erwägen der Verwendung der Regionen **USA, Osten** oder **USA, Osten 2** für die Bereitstellung Ihrer Ressourcen

1. Wählen Sie auf der **Registerkarte "Überprüfen+ Erstellen** " des **Blatts "Näherungsplatzierungsgruppen** erstellen" die Option **"Erstellen"** aus.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen wurde. Das sollte weniger als eine Minute dauern.

1. Verwenden Sie im Azure-Portal das **Textfeld "Ressourcen, Dienste und Dokumente** suchen" oben auf der Azure-Portalseite, um auf dem Blatt "Virtuelle Computer" nach dem **Blatt "Virtuelle Computer** " zu suchen und zu navigieren. Wählen Sie dann auf dem **Blatt "Virtuelle Computer** " die Option **"+Erstellen" aus ** und wählen Sie ** im Dropdownmenü die Option "Azure Virtueller Computer" ** aus.

1. Geben Sie auf dem **Blatt **Arbeitsbereich erstellen **auf der Registerkarte** Grundeinstellungen die folgenden Einstellungen an, und wählen Sie **Weiter: Disks > ** (Behalten Sie für alle anderen Einstellungen die Standardwerte bei):

    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *Wählen Sie den Namen der Ressourcengruppe aus, die Sie zuvor in diesem Vorgang verwendet haben* |
    | **Name des virtuellen Computers** | *Wählen Sie * **az12001a-vm0** aus |
    | **Region** | *Dieselbe **Azure-Region**, die Sie beim Erstellen der Näherungsplatzierungsgruppe ausgewählt haben* |
    | **Verfügbarkeitsoptionen** | *Wählen* Sie** die Verfügbarkeitsgruppe aus** |
    | **Verfügbarkeitsgruppe** | *ein neuer Verfügbarkeitssatz namens***az12001a-avset***mit 2 Fehlerdomänen und 5 Updatedomänen* |
    | **Sicherheitstyp** | *Wählen*** Sie Standard aus** |
    | **Image** | *Wählen Sie***SUSE Enterprise Linux for SAP 15 SP3 - BYOS - x64 Gen 2 aus** |
    | **Mit Azure Spot-Rabatt ausführen** | **Nein** |
    | **Größe** | **Standard D4s v3** |
    | **Authentifizierungstyp** | **Kennwort** |
    | **Benutzername** | **student** |
    | **Kennwort** | beliebiges komplexes Kennwort Ihrer Wahl |
   
    > **Hinweis:** Stellen Sie sicher, dass Sie sich das Kennwort merken, das Sie während der Bereitstellung angegeben haben. Sie benötigen diese später in diesem Lab.

    > **Hinweis:** Um das Bild zu finden, klicken Sie auf den **Link "Alle Bilder** anzeigen", geben **Sie** im Suchtextfeld im Suchtextfeld **SUSE Enterprise Linux** ein, und klicken Sie in der Ergebnisliste auf **SUSE Enterprise Linux für SAP 15 SP3 – BYOS**, und wählen Sie** "Generation 2 " ** aus.

1. Geben Sie auf dem auf der **Registerkarte **Virtuellen **Computer erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Netzwerk->** (alle anderen Einstellungen mit ihrem Standardwert belassen):

    | Einstellung | Wert |
    |   --    |  --   |
    | **Typ des Betriebssystemdatenträgers** | **Premium SSD (lokal redundanter Speicher)**  |
    | **Schlüsselverwaltung** | **Plattformseitig verwalteter Schlüssel** |

1. Geben Sie auf der Registerkarte **Networking** des Blatts **"Erstellen eines virtuellen Computers** " die folgenden Einstellungen an und wählen Sie **Weiter aus: Verwaltung >** (alle anderen Einstellungen mit ihrem Standardwert belassen):

    | Einstellung | Wert |
    |   --    |  --   |
    | **Virtuelles Netzwerk** | *wählen Sie* **Neu** *erstellen und ein neues virtuelles Netzwerk namens* **az12001a-RG-vnet-** erstellen, fahren Sie mit den nächsten Schritten unter "Erstellen eines virtuellen Netzwerks" fort.  |
    | **Adressraum** | *Legen Sie den Adressraum des neuen virtuellen Netzwerks auf***192.168.0.0/20 fest** |
    | **Subnetzname** | **subnet-0** |
    | **Subnetzadressbereich** | **192.168.0.0/24**, wählen Sie "OK" aus, um mit "Erstellen eines virtuellen Computers" fortzufahren.|
    | **Öffentliche IP-Adresse** | **none** |
    | **NIC-Netzwerksicherheitsgruppe** | **Erweitert**  |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Ein** |
    | **Optionen für den Lastenausgleich** | **Keine** |
    
    > **Hinweis:** Dieses Bild verfügt über vorkonfigurierte NSG-Regeln.

1. Klicken Sie auf **Verwaltung** und geben Sie auf der Registerkarte **Virtuellen Computer erstellen**die folgenden Einstellungen ein und wählen Sie ** Weiter aus: Überwachung >** (behalten Sie alle anderen Einstellungen mit ihrem Standardwert bei):
   
   | Einstellung | Wert |
   |   --    |  --   |
   | **Aktivieren einer systemseitig zugewiesenen verwalteten Identität** | **Deaktiviert** |
   | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
   | **Kostenlosen Basic-Plan aktivieren** | **Nein**  |

   > **Hinweis:** Der **grundlegende Plan für die kostenlose** Einstellung ist nicht verfügbar, wenn Sie Microsoft Defender für Cloud bereits in Ihrem Abonnement aktiviert haben.

1. Wählen Sie auf der **Registerkarte "Überwachung** " des **Blatts "Erstellen eines virtuellen Computers** " die Option "Weiter" aus **: Erweitert >** (alle Einstellungen mit ihrem Standardwert belassen)

1. Geben Sie auf der **Registerkarte "Erweitert** " des **Blatts "Erstellen eines virtuellen Computers** " die folgenden Einstellungen an, und wählen Sie **"Überprüfen und erstellen"** aus (belassen Sie alle anderen Einstellungen mit ihrem Standardwert):

   | Einstellung | Wert |
   |   --    |  --   |
   | **Näherungsplatzierungsgruppe** | **az12001a-ppg** |

1. Klicken Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Virtuellen Computer erstellen** auf **Erstellen**.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen wurde. Dieser Vorgang sollte weniger als 3 Minuten dauern.

1. Verwenden Sie im Azure-Portal das **Textfeld "Ressourcen, Dienste und Dokumente** suchen" oben auf der Azure-Portalseite, um auf dem Blatt "Virtuelle Computer" nach dem **Blatt "Virtuelle Computer** " zu suchen und zu navigieren. Wählen Sie dann auf dem **Blatt "Virtuelle Computer** " die Option **"+Erstellen" aus ** und wählen Sie ** im Dropdownmenü die Option "Azure Virtueller Computer" ** aus.

1. Geben Sie auf dem **Blatt **Arbeitsbereich erstellen **auf der Registerkarte** Grundeinstellungen die folgenden Einstellungen an, und wählen Sie **Weiter: Disks > ** (Behalten Sie für alle anderen Einstellungen die Standardwerte bei):
   
    | Einstellung | Wert |
    |   --    |  --   |
    | **Abonnement** | *der Name Ihres Azure-Abonnements*  |
    | **Ressourcengruppe** | *Wählen Sie den Namen der Ressourcengruppe aus, die Sie zuvor in diesem Vorgang verwendet haben* |
    | **Name des virtuellen Computers** | *Wählen Sie* **az12001a-vm1** |
    | **Region** | *dieselbe Azure-Region, die Sie beim Erstellen der Näherungsplatzierungsgruppe ausgewählt haben* |
    | **Verfügbarkeitsoptionen** | *Wählen* Sie** die Verfügbarkeitsgruppe aus** |
    | **Verfügbarkeitsgruppe** | **az12001a-avset** |
    | **Sicherheitstyp** | *Wählen*** Sie Standard aus** |
    | **Image** | *Wählen Sie* ***SUSE Enterprise Linux for SAP 15 SP3 - BYOS - x64 Gen 2 aus** |
    | **Mit Azure Spot-Rabatt ausführen** | **Nein** |
    | **Größe** | **Standard D4s v3** |
    | **Authentifizierungstyp** | **Kennwort** |
    | **Benutzername** | **student** |
    | **Kennwort** | dasselbe Kennwort, das Sie während der ersten Bereitstellung angegeben haben |
   
   > **Hinweis:** Um das Bild zu finden, klicken Sie auf den **Link "Alle Bilder** anzeigen", geben **Sie** im Suchtextfeld im Suchtextfeld **SUSE Enterprise Linux** ein, und klicken Sie in der Ergebnisliste auf **SUSE Enterprise Linux für SAP 15 SP3 – BYOS**, und wählen Sie** "Generation 2 " ** aus.

1. Geben Sie auf dem auf der **Registerkarte **Virtuellen **Computer erstellen** die folgenden Einstellungen an und wählen Sie **Weiter: Netzwerk->** (alle anderen Einstellungen mit ihrem Standardwert belassen):

    | Einstellung | Wert |
    |   --    |  --   |
    | **Typ des Betriebssystemdatenträgers** | **SSD Premium**  |
    | **Schlüsselverwaltung** | **Plattformseitig verwalteter Schlüssel** |

1. Geben Sie auf der Registerkarte **Networking** des Blatts **"Erstellen eines virtuellen Computers** " die folgenden Einstellungen an und wählen Sie **Weiter aus: Verwaltung >** (alle anderen Einstellungen mit ihrem Standardwert belassen):
    
    | Einstellung | Wert |
    |   --    |  --   |
    | **Virtuelles Netzwerk** | **az12001a-RG-vnet** |
    | **Subnetz** | **subnet-0 (192.168.0.0/24)** |
    | **Öffentliche IP-Adresse** | **none** |
    | **NIC-Netzwerksicherheitsgruppe** | **Erweitert**  |
    | **Aktivieren des beschleunigten Netzwerkbetriebs** | **Ein** |
    | **Optionen für den Lastenausgleich** | **Keine** |

   > **Hinweis:** Dieses Bild verfügt über vorkonfigurierte NSG-Regeln.

1. Klicken Sie auf **Verwaltung** und geben Sie auf der Registerkarte **Virtuellen Computer erstellen**die folgenden Einstellungen ein und wählen Sie ** Weiter aus: Überwachung >** (behalten Sie alle anderen Einstellungen mit ihrem Standardwert bei):
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Aktivieren einer systemseitig zugewiesenen verwalteten Identität** | **Deaktiviert** |
   | **Automatisches Herunterfahren aktivieren** | **Deaktiviert** |
   | **Kostenlosen Basic-Plan aktivieren** | **Nein**  |

   > **Hinweis:** Der **grundlegende Plan für die kostenlose** Einstellung ist nicht verfügbar, wenn Sie bereits den Azure Security Center-Plan ausgewählt haben.

1.  Wählen Sie auf der **Registerkarte "Überwachung** " des **Blatts "Erstellen eines virtuellen Computers** " die Option "Weiter" aus **: Erweitert >** (alle Einstellungen mit ihrem Standardwert belassen)

1.  Geben Sie auf der **Registerkarte "Erweitert** " des **Blatts "Erstellen eines virtuellen Computers** " die folgenden Einstellungen an, und wählen Sie **"Überprüfen und erstellen"** aus (belassen Sie alle anderen Einstellungen mit ihrem Standardwert):
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Näherungsplatzierungsgruppe** | **az12001a-ppg** |

1.  Klicken Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Virtuellen Computer erstellen** auf **Erstellen**.

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen wurde. Dieser Vorgang sollte weniger als 3 Minuten dauern.


### Aufgabe 2: Erstellen und Konfigurieren von Azure-VMs-Datenträgern

1. Starten Sie im Azure-Portal eine Bash-Sitzung in Cloud Shell. 

   > **Hinweis:** Wenn Sie Cloud Shell zum ersten Mal im aktuellen Azure-Abonnement starten, werden Sie aufgefordert, eine Azure-Dateifreigabe zu erstellen, um Cloud Shell-Dateien beizubehalten. Akzeptieren Sie in diesem Falls die Standardwerte, was dazu führt, dass ein Speicherkonto in einer automatisch generierten Ressourcengruppe erstellt wird.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `RESOURCE_GROUP_NAME` auf den Namen der Ressourcengruppe festzulegen, die die Ressourcen enthält, die Sie im vorherigen Vorgang bereitgestellt haben:

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die erste Gruppe von 8 verwalteten Datenträgern zu erstellen, die Sie an die erste Azure-VM anfügen, die Sie in der vorherigen Aufgabe bereitgestellt haben:

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die zweite Gruppe von 8 verwalteten Datenträgern zu erstellen, die Sie an die zweite Azure-VM anfügen, die Sie in der vorherigen Aufgabe bereitgestellt haben:

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Navigieren Sie im Azure-Portal zum Blatt der ersten Azure-VM, die Sie in der vorherigen Aufgabe bereitgestellt haben (**az12001a-vm0**).

1. Navigieren Sie vom **Blatt "az12001a-vm0** " zum **Blatt "az12001a-vm0 \| Disks** ".

1. Wählen Sie auf dem **Blatt "az12001a-vm0 \| Disks** " die Option **"Vorhandene Datenträger** anfügen" aus, und fügen Sie den Datenträger mit den folgenden Einstellungen an az12001a-vm0 an:
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Name des Datenträgers** | **az12001a-vm0-DataDisk0** |
   | **Ressourcengruppe** | *Wählen Sie den Namen der Ressourcengruppe aus, die Sie zuvor in diesem Vorgang verwendet haben* |
   | **HOSTZWISCHENSPEICHERUNG** | **Schreibgeschützt** |

2. Wiederholen Sie den vorherigen Schritt, um die verbleibenden 7 Datenträger mit dem Präfix **az12001a-vm0-DataDisk** (für die Summe von 8) anzufügen. Weisen Sie die LUN-Nummer zu, die dem letzten Zeichen des Datenträgernamens entsprechen. Legen Sie DIE HOSTZWISCHENSPEICHERÚNG des Datenträgers mit LUN **1** auf **schreibgeschützt** fest und legen Sie für alle verbleibenden HOSTZWISCHENSPEICHERUNG auf **"Keine"** fest.

3. Speichern Sie die Änderungen. 

4. Navigieren Sie im Azure-Portal zum Blatt der zweiten Azure-VM, die Sie in der vorherigen Aufgabe **(az12001a-vm1**) bereitgestellt haben.

5. Navigieren Sie vom **Blatt "az12001a-vm1"** zum **Blatt "az12001a-vm1 \| Disks"**.

6. Fügen Sie auf dem **Blatt "az12001a-vm1 \| Disks** " Datenträger mit den folgenden Einstellungen an az12001a-vm1 an:
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Name des Datenträgers** | **az12001a-vm1-DataDisk0** |
   | **Ressourcengruppe** | *Wählen Sie den Namen der Ressourcengruppe aus, die Sie zuvor in diesem Vorgang verwendet haben* |
   | **HOSTZWISCHENSPEICHERUNG** | **Schreibgeschützt** |

7. Wiederholen Sie den vorherigen Schritt, um die verbleibenden 7 Datenträger mit dem Präfix **az12001a-vm1-DataDisk** (für die Summe von 8) anzufügen. Weisen Sie die LUN-Nummer zu, die dem letzten Zeichen des Datenträgernamens entsprechen. Legen Sie DIE HOSTZWISCHENSPEICHERÚNG des Datenträgers mit LUN **1** auf **schreibgeschützt** fest und legen Sie für alle verbleibenden HOSTZWISCHENSPEICHERUNG auf **"Keine"** fest.

8. Speichern Sie die Änderungen. 

#### Aufgabe 3: Bereitstellen von Azure Bastion 

> **Hinweis:** Azure Bastion ermöglicht die Verbindungsherstellung mit den virtuellen Azure-Computern ohne öffentliche Endpunkte, die Sie in der vorherigen Aufgabe dieser Übung bereitgestellt haben, und bietet gleichzeitig Schutz vor Brute-Force-Angriffen, die auf Anmeldeinformationen auf Betriebssystemebene abzielen.

> **Hinweis:** Stellen Sie sicher, dass bei Ihrem Browser die Popupfunktion aktiviert ist.

1. Öffnen Sie im Browserfenster mit dem Azure-Portal einen weiteren Tab, und navigieren Sie zum [**Azure-Portal**](https://portal.azure.com).
1. Öffnen Sie im Azure-Portal den Bereich **Cloud Shell**, indem Sie rechts neben dem Textfeld für die Suche das Symbolleistensymbol auswählen.
1. Führen Sie aus der PowerShell-Sitzung im Cloud Shell-Bereich den folgenden Code aus, um ein Subnetz namens **AzureBastionSubnet** zum virtuellen Netzwerk namens **az12001a-25-vnet** hinzuzufügen, das Sie in einer vorherigen Übung erstellt haben:

   ```powershell
   $resourceGroupName = 'az12001a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az12001a-RG-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 192.168.15.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Schließen Sie den Cloud Shell-Bereich.
1. Suchen Sie im Azure-Portal nach **Bastions**, wählen Sie diese Option aus, und wählen Sie auf dem Blatt **Bastions** die Option **+ Erstellen** aus.
1. Geben Sie auf der Registerkarte **Grundeinstellungen** des Blatts **Bastion erstellen** die folgenden Einstellungen an, und wählen Sie **Überprüfen + erstellen** aus:

   |Einstellung|Wert|
   |---|---|
   |Subscription|Der Name des Azure-Abonnements, das Sie in diesem Lab verwenden.|
   |Resource group|**az12001a-RG**|
   |Name|**az12001a-bastion**|
   |Region|Die gleiche Azure-Region, in der Sie die Ressourcen in der vorherigen Aufgabe dieser Übung bereitgestellt haben|
   |Tarif|**Grundlegend**|
   |Virtuelles Netzwerk|**az12001a-RG-vnet**|
   |Subnetz|**AzureBastionSubnet (192.168.15.0/24)**|
   |Öffentliche IP-Adresse|**Neu erstellen**|
   |Name der öffentlichen IP-Adresse|**az12001a-RG-vnet-ip**|

1. Wählen Sie auf der Registerkarte **Überprüfen + erstellen** des Blatts **Bastion erstellen** die Option **Erstellen** aus:

   > **Hinweis:** Warten Sie, bis die Bereitstellung abgeschlossen ist, bevor Sie mit der nächsten Aufgabe dieser Übung fortfahren. Die Bereitstellung kann ungefähr 5 Minuten dauern.


> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie Azure-Computerressourcen bereitgestellt, die erforderlich sind, um hoch verfügbare SAP HANA-Bereitstellungen zu unterstützen.


## Übung 2: Konfigurieren des Betriebssystems von Azure-VMs unter Linux zur Unterstützung einer hochverfügbaren SAP HANA Installation

Dauer: 30 Minuten

In dieser Übung konfigurieren Sie das Betriebssystem und den Speicher auf Azure-VMs, auf denen SUSE Linux Enterprise Server ausgeführt wird, um gruppierte Installationen von SAP HANA aufzunehmen.

### Aufgabe 1: Herstellen einer Verbindung mit virtuellen Azure Linux-Computern

1. Suchen Sie auf Ihrem Labcomputer im Azure-Portal nach **Virtuelle Computer** und wählen Sie diese Option aus. Wählen Sie anschließend auf dem Blatt **Virtuelle Computer** den Eintrag **az12001a-cl-vm0** aus. Dadurch wird das **Blatt az12001a-vm0** geöffnet.

1. Wählen Sie auf dem **Blatt "az12001a-vm0"** die Option **"Verbinden" **aus, wählen Sie** im Dropdownmenü "Verbinden über Bastion**" auf der **Registerkarte "Bastion** " der **az12001a-vm0**aus, lassen Sie den **Authentifizierungstyp** auf **"VM-Kennwort**" festgelegt, geben Sie die Anmeldeinformationen an, die Sie beim Bereitstellen des **virtuellen Computers az12001a-vm0** festgelegt haben, lassen Sie das Kontrollkästchen **"In neuer Browserregisterkarte öffnen"** aktiviert, und wählen Sie "Verbinden" **aus**:

1. Wiederholen Sie die beiden vorherigen Schritte, um eine Verbindung über Bastion mit der **Azure-VM az12001a-vm1** herzustellen.

### Aufgabe 2: Konfigurieren des Speichers von Azure-VMs unter Linux

1. Führen Sie in der Bastion-Sitzung auf die **Azure-VM az12001a-vm0** den folgenden Befehl aus, um Rechte zu erhöhen: 

   ```cli
   sudo su -
   ```

1. Führen Sie den folgenden Befehl aus, um die Zuordnung zwischen den neu angeschlossenen Geräten und deren LUN-Nummern zu identifizieren:
   
   ```cli
   lsscsi
   ```

1. Erstellen Sie physische Volumes für 6 (von 8) Datenträgern, indem Sie Folgendes ausführen:
   
   ```cli
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. Erstellen Sie Volumengruppen, indem Sie Folgendes ausführen:
   
   ```cli
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. Erstellen Sie logische Volumes, indem Sie Folgendes ausführen:

   ```cli
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **Hinweis:** Wir erstellen ein einzelnes logisches Volume pro Volumegruppe

1. Formatieren Sie die logischen Volumes, indem Sie Folgendes ausführen:

   ```cli
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **Hinweis:** Ab SUSE Linux Enterprise Server 12 haben Sie die Möglichkeit, das neue On-Disk-Format (v5) des XFS-Dateisystems zu verwenden, das automatische Prüfsummen von XFS-Metadaten, Dateitypunterstützung und einen erhöhten Grenzwert für die Anzahl der Zugriffssteuerungslisten pro Datei bietet. Das neue Format wird automatisch angewendet, wenn YaST zum Erstellen der XFS-Dateisysteme verwendet wird. Um aus Kompatibilitätsgründen ein XFS-Dateisystem im älteren Format zu erstellen, verwenden Sie den Befehl mkfs.xfs ohne die `-m crc=1` Option. 

1. Partitionieren Sie den **Datenträger "/dev/sdi** ", indem Sie Folgendes ausführen:

   ```cli
   fdisk /dev/sdi
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie in Folge, `n`, `p` `1` (gefolgt von der **EINGABETASTE** jedes Mal) zweimal die **EINGABETASTE** ein, und geben `w` Sie dann ein, um den Schreibvorgang abzuschließen.

1. Partitionieren Sie den **Datenträger "/dev/sdj** ", indem Sie Folgendes ausführen:

   ```cli
   fdisk /dev/sdj
   ```

1. Wenn Sie dazu aufgefordert werden, geben Sie in Folge, `n`, `p` `1` (gefolgt von der **EINGABETASTE** jedes Mal) zweimal die **EINGABETASTE** ein, und geben `w` Sie dann ein, um den Schreibvorgang abzuschließen.

1. Formatieren Sie die neu erstellte Partition, indem Sie ausgeführt werden (geben Sie `y` die **EINGABETASTE** ein, wenn Sie zur Bestätigung aufgefordert werden):

   ```cli
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. Erstellen Sie die Verzeichnisse, die als Bereitstellungspunkte dienen, indem Sie Folgendes ausführen:

   ```cli
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. Zeigen Sie die IDs logischer Volumes an, indem Sie Folgendes ausführen:

   ```cli
   blkid
   ```

   > **Hinweis:** Identifizieren Sie die **UUID-Werte **, die den neu erstellten Volumegruppen und Partitionen zugeordnet sind, einschließlich** /dev/sdi **(für** /hana/shared **) und**dev/sdj **(für **/usr/sap** verwendet werden).


1. Öffnen Sie **/etc/fstab** im vi-Editor (Sie können einen anderen Editor verwenden), indem Sie Folgendes ausführen:

   ```cli
   vi /etc/fstab
   ```

1. Fügen Sie im Editor die folgenden Einträge zu **/etc/fstab** hinzu (wobei `\<UUID of /dev/vg\_hana\_data-hana\_data\>`,`\<UUID of /dev/vg\_hana\_log-hana\_log\>`, `\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`, `\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>`und `\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>`, die IDs darstellen, die Sie im vorherigen Schritt identifiziert haben):

   ```cli
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Stellen Sie die neuen Volumes bereit, indem Sie Folgendes ausführen:

   ```cli
   mount -a
   ```

1. Überprüfen Sie, ob die Bereitstellung erfolgreich war, indem Sie Folgendes ausführen:

   ```cli
   df -h
   ```

1. Wechseln Sie zur Bastion-Sitzung zu az12001a-vm1, und wiederholen Sie alle Schritte in diesen Aufgaben, um den Speicher auf **az12001a-vm1**zu konfigurieren.


### Aufgabe 3: Aktivieren des knotenübergreifenden Kennworts ohne SSH-Zugriff

1. Generieren Sie in der Bastion-Sitzung an der **az12001a-vm0** Azure VM den passphrasenlosen SSH-Schlüssel, indem Sie Folgendes ausführen:

   ```cli
   ssh-keygen -tdsa
   ```

1. Wenn Sie dazu aufgefordert werden, **drücken Sie** dreimal den öffentlichen Schlüssel, und zeigen Sie dann die öffentliche Taste an, indem Sie Folgendes ausführen: 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. Kopieren Sie den Wert des Schlüssels in die Zwischenablage.

1. Wechseln Sie zur Bastion-Sitzung zur **Azure-VM az12001a-vm1** und erstellen Sie eine Datei **/root/.ssh/autorisierte\_Schlüssel** im vi-Editor (Sie können einen anderen Editor verwenden), indem Sie Folgendes ausführen:

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. Fügen Sie im Editorfenster den Schlüssel ein, den Sie auf az12001a-vm0 generiert haben.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Generieren Sie in der Bastion-Sitzung auf der **Azure-VM az12001a-vm1** passphrase-weniger SSH-Schlüssel, indem Sie Folgendes ausführen:

   ```cli
   ssh-keygen -tdsa
   ```

1. Wenn Sie dazu aufgefordert werden, **drücken Sie** dreimal den öffentlichen Schlüssel, und zeigen Sie dann die öffentliche Taste an, indem Sie Folgendes ausführen: 

   ```cli
   cat /root/.ssh/id_dsa.pub
   ```

1. Kopieren Sie den Wert des Schlüssels in die Zwischenablage.

1. Wechseln Sie zur Bastion-Sitzung zur **az12001a-vm0** Azure VM und erstellen Sie eine Datei **/root/.ssh/autorisierte\_Schlüssel** im vi-Editor (Sie können jeden anderen Editor verwenden), indem Sie Folgendes ausführen:

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. Fügen Sie im Editorfenster den Schlüssel ein, den Sie auf az12001a-vm1 generiert haben.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Generieren Sie in der Bastion-Sitzung an der **az12001a-vm0** Azure VM den passphrasenlosen SSH-Schlüssel, indem Sie Folgendes ausführen:

   ```cli
   ssh-keygen -t rsa
   ```

1. Wenn Sie dazu aufgefordert werden, **drücken Sie** dreimal den öffentlichen Schlüssel, und zeigen Sie dann die öffentliche Taste an, indem Sie Folgendes ausführen: 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. Kopieren Sie den Wert des Schlüssels in die Zwischenablage.

1. Wechseln Sie zur Bastion-Sitzung zur **Azure-VM az12001a-vm1** und öffnen Sie die Datei **"/root/.ssh/authorized\_"-Schlüssel** im vi-Editor (Sie können jeden anderen Editor verwenden), indem Sie Folgendes ausführen:

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. Fügen Sie im Editorfenster beginnend mit einer neuen Zeile den Schlüssel ein, den Sie auf az12001a-vm0 generiert haben.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Generieren Sie in der Bastion-Sitzung auf der **Azure-VM az12001a-vm1** passphrase-weniger SSH-Schlüssel, indem Sie Folgendes ausführen:

   ```cli
   ssh-keygen -t rsa
   ```

1. Wenn Sie dazu aufgefordert werden, **drücken Sie** dreimal den öffentlichen Schlüssel, und zeigen Sie dann die öffentliche Taste an, indem Sie Folgendes ausführen: 

   ```cli
   cat /root/.ssh/id_rsa.pub
   ```

1. Kopieren Sie den Wert des Schlüssels in die Zwischenablage.

1. Wechseln Sie zur Bastion-Sitzung zur **az12001a-vm0** Azure VM, und öffnen Sie die Datei **/root/.ssh/autorisierte\_Schlüssel** im vi-Editor (Sie können einen anderen Editor verwenden), indem Sie Folgendes ausführen:

   ```cli
   vi /root/.ssh/authorized_keys
   ```

1. Fügen Sie im Editorfenster beginnend mit einer neuen Zeile den Schlüssel ein, den Sie auf az12001a-vm1 generiert haben.

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Öffnen Sie in der Bastion-Sitzung auf der **azure-VM az12001a-vm0** die Datei **/etc/ssh/sshd\_config** im vi-Editor (Sie können einen anderen Editor verwenden), indem Sie Folgendes ausführen:

   ```cli
   vi /etc/ssh/sshd_config
   ```

1. Suchen Sie in der **Konfigurationsdatei "/etc/sshd\_"** die **Einträge "PermitRootLogin** " und **"AuthorizedKeysFile** " und konfigurieren Sie sie wie folgt (entfernen Sie bei Bedarf das führende **#** Zeichen:

   ```cli
   PermitRootLogin yes
   AuthorizedKeysFile  /root/.ssh/authorized_keys
   ```

1. Speichern Sie die Änderungen, und schließen Sie den Editor.

1. Starten Sie innerhalb der Bastion-Sitzung auf der **az12001a-vm0** Azure VM sshd daemon neu, indem Sie Folgendes ausführen:

   ```cli
   systemctl restart sshd
   ```

1. Wiederholen Sie die vorherigen vier Schritte auf az12001a-vm1.

1. Um zu überprüfen, ob die Konfiguration erfolgreich war, richten Sie in der Bastion-Sitzung an die **az12001a-vm0 Azure VM eine** SSH-Sitzung als **Stamm** von az12001a-vm0 zu az12001a-vm1 ein, indem Sie Folgendes ausführen: 

   ```cli
   ssh root@az12001a-vm1
   ```

1. Wenn Sie gefragt werden, ob Sie die Verbindung fortsetzen möchten, geben Sie `yes` ein und drücken Sie die **EINGABETASTE**. 

1. Stellen Sie sicher, dass Sie nicht zur Eingabe des Kennworts aufgefordert werden.

1. Schließen Sie die SSH-Sitzung von az12001a-vm0 auf az12001a-vm1, indem Sie Folgendes ausführen: 

   ```cli
   exit
   ```

1. Melden Sie sich von az12001a-vm0 ab, indem Sie die folgenden zwei Mal ausführen:

   ```cli
   exit
   ```

1. Um zu überprüfen, ob die Konfiguration erfolgreich war, richten Sie in der Bastion-Sitzung auf **az12001a-vm1**eine SSH-Sitzung als **Stamm** von az12001a-vm1 zu az12001a-vm0 ein, indem Sie Folgendes ausführen: 

   ```cli
   ssh root@az12001a-vm0
   ```

1. Wenn Sie gefragt werden, ob Sie die Verbindung fortsetzen möchten, geben Sie `yes` ein und drücken Sie die **EINGABETASTE**. 

1. Stellen Sie sicher, dass Sie nicht zur Eingabe des Kennworts aufgefordert werden.

1. Schließen Sie die SSH-Sitzung von az12001a-vm1 auf az12001a-vm0, indem Sie Folgendes ausführen: 

   ```cli
   exit
   ```

1. Melden Sie sich von az12001a-vm1 ab, indem Sie die folgenden zwei Mal ausführen:

   ```cli
   exit
   ```

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie das Betriebssystem von Azure-VMs unter Linux konfiguriert, um eine hoch verfügbare SAP HANA-Installation zu unterstützen


## Übung 3: Bereitstellen von Azure-Netzwerkressourcen, die zur Unterstützung von hochverfügbaren SAP HANA-Bereitstellungen erforderlich sind

Dauer: 30 Minuten

In dieser Übung implementieren Sie Azure Load Balancers, um gruppierte Installationen von SAP HANA aufzunehmen.


### Aufgabe 1: Konfigurieren Sie Azure-VMs, um das Einrichten des Lastenausgleichs zu erleichtern.

1. Navigieren Sie im Azure-Portal zum Blatt der **az12001a-vm0** Azure VM.

1. Navigieren Sie vom **Az12001a-vm0-Blatt** zum **Az12001a-vm0 \| Networking-Blatt**. 

1. Wählen Sie auf dem **Az12001a-vm0 \| Networking-Blatt** den Eintrag aus, der die Netzwerkschnittstelle der az12001a-vm0 darstellt. 

1. Navigieren Sie vom Blatt der Netzwerkschnittstelle des az12001a-vm0 zum Blatt "IP-Konfigurationen", und zeigen Sie von dort aus das **Blatt "ipconfig1** " an.

1. Legen Sie auf dem Blatt **ipconfig1** die private IP-Adresszuweisung auf **Statisch** fest, und speichern Sie die Änderung.

1. Navigieren Sie im Azure-Portal zum Blatt der **Azure-VM az12001a-vm1**.

1. Navigieren Sie vom **Az12001a-vm1-Blatt** zum **Az12001a-vm1 \| Networking** Blatt. 

1. Navigieren Sie vom **Az12001a-vm1 \| Networking-Blatt** zur Netzwerkschnittstelle der az12001a-vm1. 

1. Navigieren Sie vom Blatt der Netzwerkschnittstelle des az12001a-vm1 zum Blatt "IP-Konfigurationen", und zeigen Sie von dort aus das **Blatt "ipconfig1** " an.

1. Legen Sie auf dem Blatt **ipconfig1** die private IP-Adresszuweisung auf **Statisch** fest, und speichern Sie die Änderung.


### Aufgabe 2: Erstellen und Konfigurieren von Azure Load Balancers für eingehenden Datenverkehr

1. Verwenden Sie im Azure-Portal oben auf der Azure-Portalseite das **Textfeld "Ressourcen, Dienste und Dokumente** suchen und navigieren" zum Blatt" **Lastenausgleich"**, und wählen Sie auf dem **Blatt "Lastenausgleich"** die Option **"+Erstellen**" aus.

1. Geben Sie auf der **Registerkarte **"Grundlagen" des Blatts ** "Lastenausgleich** erstellen" die folgenden Einstellungen an, und wählen Sie **"Überprüfen+ erstellen " aus ** (belassen Sie andere Personen mit ihren Standardwerten):
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Abonnement** | *der Name Ihres Azure-Abonnements* |
   | **Ressourcengruppe** | *wählen Sie den Namen der Ressourcengruppe aus, die Sie zuvor in dieser Übung verwendet haben* |
   | **Name** | **az12001a-lb0** |
   | **Region** | *dieselbe Azure-Region, in der Sie Azure-VMs in der ersten Übung dieses Labors bereitgestellt haben* |
   | **SKU** | **Standard** |
   | **Typ** | **Intern** |

1. Klicken Sie auf **Next: Front-End-IP-Konfiguration** Klicken Sie auf dem **Front-End-IP-Konfigurationsbildschirm** auf **"Front-End-IP-Konfiguration** hinzufügen", und klicken Sie dann auf **"Hinzufügen"**.
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Name** | **frontend1** |
   | **Virtuelles Netzwerk** | **az12001a-RG-vnet** |
   | **Subnetz** | **subnet-0** |
   | **IP-Adresszuweisung** | **Statisch** |
   | **IP-Adresse** | **192.168.0.240** |
   | **Verfügbarkeitszone** | **Zonenredundant** |

1. Klicken Sie auf **Überprüfen und erstellen** und dann auf **Erstellen**.

   > **Hinweis:** Warten Sie, bis das Lastenausgleichsmodul bereitgestellt wird. Das sollte weniger als eine Minute dauern. 

1. Navigieren Sie im Azure-Portal zum Blatt, auf dem die Eigenschaften des neu bereitgestellten **az12001a-lb0-Lastenausgleichsgeräts** angezeigt werden. 

1. Wählen Sie auf dem **Blatt az12001a-lb0**** Back-End-Pools **aus, wählen Sie **+Hinzufügen **aus, und geben Sie im**Add-Back-End-Pool **die folgenden Einstellungen an (lassen Sie andere mit ihren Standardwerten):
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Name** | **az12001a-lb0-bepool** |
   | **Virtuelles Netzwerk** | **az12001a-RG-vnet** |
   | **Konfiguration des Back-End-Pools** | **IP-Adresse** |
   | **IP-Adresse** | **192.168.0.4** Ressourcenname: **az12001a-vm0** |
   | **IP-Adresse** | **192.168.0.5** Ressourcenname: **az12001a-vm1** |

1. Wählen Sie auf dem **Blatt az12001a-lb0** die Option **"Integritätssonden** " und **"Hinzufügen" ** aus, und geben Sie auf dem Blatt **"Integritätssonde hinzufügen" ** die folgenden Einstellungen an (lassen Sie andere mit ihren Standardwerten):
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Name** | **az12001a-lb0-hprobe** |
   | **Protokoll** | **TCP** |
   | **Port** | **62500** |
   | **Intervall** | **5** *Sekunden* |
   | **Fehlerhafter Schwellenwert** | **Zwei *** aufeinanderfolgende Fehler* |

1. Wählen Sie auf dem **Blatt az12001a-lb0** die Option **Lastenausgleichsregeln**aus, wählen Sie **+Hinzufügen aus, und geben Sie auf dem Blatt** "Lastenausgleichsregel hinzufügen"** die folgenden Einstellungen ** an (lassen Sie andere mit ihren Standardwerten):
    
   | Einstellung | Wert |
   |   --    |  --   |
   | **Name** | **az12001a-lb0-lbruleAll** |
   | **IP-Version** | **IPv4** |
   | **Frontend IP address** (Front-End-IP-Adresse) | **192.168.0.240 (LoadBalancerFrontEnd)** |
   | **Hochverfügbarkeitsports** | **Enabled** |
   | **Back-End-Pool** | **az12001a-lb0-bepool (2 virtual machines)** |
   | **Integritätstest** | **az12001a-lb0-hprobe (TCP:62500)** |
   | **Sitzungspersistenz** | **None** |
   | **Leerlaufzeitüberschreitung (Minuten)** | **4** |
   | **TCP-Zurücksetzung** | **Disabled** |
   | **Floating IP (Direct Server Return)** | **Enabled** |

### Aufgabe 3: Erstellen und Konfigurieren von Azure Load Balancers für ausgehenden Datenverkehr

1. Starten Sie im Azure-Portal eine Bash-Sitzung in Cloud Shell. 

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `RESOURCE_GROUP_NAME` auf den Namen der Ressourcengruppe festzulegen, die die Ressourcen enthält, die Sie in der ersten Übung dieses Labs bereitgestellt haben:

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die öffentliche IP-Adresse zu erstellen, die vom zweiten Lastenausgleichsmodul verwendet werden soll:

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um das zweite Lastenausgleichsmodul zu erstellen:

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um die Ausgangsregel des zweiten Lastenausgleichs zu erstellen:

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. Schließen Sie den Cloud Shell-Bereich.

1. Navigieren Sie im Azure-Portal zum Blatt, auf dem die Eigenschaften der neu erstellten Azure Load Balancer **az12001a-lb1**angezeigt werden.

1. Klicken Sie auf dem **Blatt az12001a-lb1** auf **Back-End-Pools**.

1. Klicken Sie auf dem **Blatt az12001a-lb1-Back-End-Pools \|** auf **az12001a-lb1-bepool**.

1. Geben Sie auf dem **Blatt az12001a-lb1-bepool** die folgenden Einstellungen an, und klicken Sie auf " **Speichern**":
   
   | Einstellung | Wert |
   |   --    |  --   |
   | **Virtuelles Netzwerk** | **az12001a-rg-vnet (2 VM)** |
   | **Virtueller Computer** | **az12001a-vm0**  IP Konfiguration: **ipconfig1 (192.168.0.4)** |
   | **Virtueller Computer** | **az12001a-vm1**  IP Konfiguration: **ipconfig1 (192.168.0.5)** |

> **Result**: Nachdem Sie diese Übung abgeschlossen haben, haben Sie Azure-Netzwerkressourcen bereitgestellt, die erforderlich sind, um hoch verfügbare SAP HANA-Bereitstellungen zu unterstützen


## Übung 4: Entfernen der Labressourcen

Dauer: 10 Minuten

In dieser Übung entfernen Sie in dieser Übung bereitgestellte Ressourcen.

#### Aufgabe 1: Zu löschende Ressourcengruppen auflisten

1. Klicken Sie oben im Portal auf das Cloud Shell-Symbol, um den **Cloud Shell-Bereich** zu öffnen und wählen Sie Bash als Shell aus.

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um den Wert der Variablen `RESOURCE_GROUP_PREFIX` auf das Präfix des Namens der Ressourcengruppe festzulegen, die die in dieser Übung bereitgestellten Ressourcen enthält:

   ```cli
   RESOURCE_GROUP_PREFIX='az12001a-'
   ```

1. Führen Sie im Cloud Shell-Bereich den folgenden Befehl aus, um alle Ressourcengruppen auflisten zu können, die Sie in dieser Übung erstellt haben:

   ```cli
   az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
   ```

1. Stellen Sie sicher, dass die Ausgabe nur die Ressourcengruppe enthält, die Sie in dieser Übung erstellt haben. Diese Ressourcengruppe mit allen Ressourcen wird im nächsten Vorgang gelöscht.

#### Aufgabe 2: Löschen der Ressourcengruppen

1. Führen Sie im Bereich Cloud Shell den folgenden Befehl aus, um die AsyncProcessor-Ressourcengruppe zu löschen.

   ```cli
   az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Schließen Sie den Cloud Shell-Bereich.

> **Result**: Durch den Abschluss dieser Übung haben Sie die in diesem Lab verwendeten Ressourcen entfernt.
