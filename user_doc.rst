.. index:: Plugins; AVM (Unterstützung für Fritz!Box usw.)
.. index:: AVM

avm
###

Changelog
=========

1.6.0
-----

- Anbindung der Smarthome Devices über AHA-Interface hinzugefügt
- Funktionen für Rufumleitungen hinzugefügt
- Plugin Parameter "index" in "avm_tam_index" umbenannt
- Code Cleanup


Konfiguration der Fritz!Box
===========================

Für die Nutzung der Informationen über Telefonereignisse muss der CallMonitor aktiviert werden. Dazu muss auf
einem direkt an die Fritz!Box angeschlossenen Telefon (Analog, ISDN S0 oder DECT) \*96#5# eingegeben werden.

Bei neueren Firmware Versionen (ab Fritz!OS v7) Muss die Anmeldung an der Box von "nur mit Kennwort" auf "Benutzername
und Kennwort umgestellt werden" und es sollte ein eigener User für das AVM Plugin auf der Fritz!Box eingerichtet werden.

Konfiguration des Plugins
=========================

Die Konfigruation des Plugins erfolgt über das Admin-Inteface.
Dafür stehen die folgenden Einstellungen zur Verfügung:

  - `username`: Required login information
  - `password`: Required login information
  - `host`: Hostname or ip address of the FritzDevice.
  - `port`: Port of the FritzDevice, typically 49433 for https or 49000 for http
  - `cycle`: timeperiod between two update cycles. Default is 300 seconds.
  - `ssl`: True or False => True will add "https", False "http" to the URLs in the plugin
  - `verify`: True or False => Turns certificate verification on or off. Typically False
  - `call_monitor`: True or False => Activates or deactivates the MonitoringService, which connects to the FritzDevice's call monitor
  - `instance`: Unique identifier for each FritzDevice / each instance of the plugin

Alternativ kann das Plugin auch manuell konfiguriert werden.

.. code:: yaml

    fb1:
        class_name: AVM
        class_path: plugins.avm
        username: ...    # optional
        password: '...'
        host: fritz.box
        port: 49443
        cycle: 300
        ssl: True    # use https or not
        verify: False    # verify ssl certificate
        call_monitor: 'True'
        call_monitor_incoming_filter: "...    ## optional, don't set if you don't want to watch only one specific number with your call monitor"
        instance: fritzbox_7490

    fb2:
        class_name: AVM
        class_path: plugins.avm
        username: ...    # optional
        password: '...'
        host: '...'
        port: 49443
        cycle: 300
        ssl: True    # use https or not
        verify: False    # verify ssl certificate
        call_monitor: 'True'
        instance: wlan_repeater_1750

Note: Depending on the FritzDevice a shorter cycle time can result in problems with CPU rating and, in consequence with the accessibility of the webservices on the device.
If cycle time is reduced, please carefully watch your device and your sh.log. In the development process, 120 Seconds also worked worked fine on the used devices.

Konfiguration des Items
=======================

Zur Konfiguration der Items stehen folgende Parameter zur Verfügung:

avm_data_type
-------------
This attribute defines supported functions that can be set for an item. Full set see plugin.yaml.
For most items, the avm_data_type can be bound to an instance via @... . Only in some points the items
are parsed as child items.

avm_incoming_allowed
--------------------
Definition der erlaubten eingehenden Rufnummer in Items vom avm_data_type `monitor_trigger`.'

avm_target_number
-----------------
Definition der erlaubten angerufenen Rufnummer in Items vom avm_data_type `monitor_trigger`.'

avm_wlan_index
--------------
Definition des Wlans ueber index: (1: 2.4Ghz, 2: 5Ghz, 3: Gaeste).'

avm_mac
-------
Definition der MAC Adresse für Items vom avm_data_type `network_device`. Nur für diese Items mandatory!'

ain
---
Definition der Aktor Identifikationsnummer (AIN)Items für smarthome Items. Nur für diese Items mandatory!'


avm_tam_index
-------------
Index für den Anrufbeantworter, normalerweise für den ersten eine "1". Es werden bis zu 5 Anrufbeantworter vom Gerät unterstützt.'


avm_deflection_index
--------------------
Index für die Rufumleitung, normalerweise für die erste eine "1".'


item_structs
============
Zur Vereinfachung der Einrichtung von Items sind für folgende Item-structs vordefiniert:
 - info  -  General Information about Fritzbox
 - monitor  -  Coll Monitor
 - tam  -  (für einen) Anrufbeantworter
 - deflection  -  (für eine) Rufumleitung
 - wan  -  WAN Items
 - wlan  -  Wireless Lan Items
 - device  -  Item eines verbundenen Gerätes
 - smarthome_general  -  Allgemeine Informationen eines DECT smarthome Devices
 - smarthome_hkr  -  spezifische Informationen eines DECT Thermostat Devices
 - smarthome_temperatur_sensor  -  spezifische Informationen eines DECT smarthome Devices mit Temperatursensor
 - smarthome_alert  -  spezifische Informationen eines DECT smarthome Devices mit Alarmfunktion
 - smarthome_switch  -  spezifische Informationen eines DECT smarthome Devices mit Schalter
 - smarthome_powermeter  -  spezifische Informationen eines DECT smarthome Devices mit Strommessung

Item Beispiel mit Verwendung der structs
----------------------------------------

.. code:: yaml

    avm:
        fritzbox:
            info:
                struct:
                  - avm.info

            reboot:
                type: bool
                visu_acl: rw
                enforce_updates: yes

            monitor:
                struct:
                  - avm.monitor

            tam:
                struct:
                  - avm.tam

            rufumleitung:
                rufumleitung_1:
                    struct:
                      - avm.deflection

                rufumleitung_2:
                    avm_deflection_index: 2
                    struct:
                      - avm.deflection

            wan:
                struct:
                  - avm.wan

            wlan:
                struct:
                  - avm.wlan

            connected_devices:
                mobile_1:
                    avm_mac: xx:xx:xx:xx:xx:xx
                    struct:
                      - avm.device

                mobile_2:
                    avm_mac: xx:xx:xx:xx:xx:xx
                    struct:
                      - avm.device

        smarthome:
            hkr_og_bad:
                type: foo
                ain: 'xxxxx xxxxxxx'
                struct:
                  - avm.smarthome_general
                  - avm.smarthome_hkr
                  - avm.smarthome_temperatur_sensor

Plugin Funktionen
=================

cancel_call
-----------
Beendet einen aktiven Anruf.

get_call_origin
---------------
Gib den Namen des Telefons zurück, das aktuell als 'call origin' gesetzt ist.

get_calllist
------------
Ermittelt ein Array mit dicts aller Einträge der Anrufliste (Attribute 'Id', 'Type', 'Caller', 'Called', 'CalledNumber', 'Name', 'Numbertype', 'Device', 'Port', 'Date',' Duration' (einige optional)).

get_contact_name_by_phone_number
--------------------------------
Durchsucht das Telefonbuch mit einer (vollständigen) Telefonnummer nach Kontakten. Falls kein Name gefunden wird, wird die Telefonnummer zurückgeliefert.

get_device_log_from_lua
-----------------------
Ermittelt die Logeinträge auf dem Gerät über die LUA Schnittstelle /query.lua?mq_log=logger:status/log.

get_device_log_from_tr064
-------------------------
Ermittelt die Logeinträge auf dem Gerät über die TR-064 Schnittstelle.

get_host_details
----------------
Ermittelt die Informationen zu einem Host an einem angegebenen Index.

get_hosts
---------
Ermittelt ein Array mit den Namen aller verbundener Hosts.

get_phone_name
--------------
Gibt den Namen eines Telefons an einem Index zurück. Der zurückgegebene Wert kann in 'set_call_origin' verwendet werden.

get_phone_numbers_by_name
-------------------------
Durchsucht das Telefonbuch mit einem Namen nach nach Kontakten und liefert die zugehörigen Telefonnummern.

is_host_active
--------------
Prüft, ob eine MAC Adresse auf dem Gerät aktiv ist. Das kann bspw. für die Umsetzung einer Präsenzerkennung genutzt werden.

reboot
------
Startet das Gerät neu.

reconnect
---------
Verbindet das Gerät neu mit dem WAN (Wide Area Network).

set_call_origin
---------------
Setzt den 'call origin', bspw. vor dem Aufruf von 'start_call'.

start_call
----------
Startet einen Anruf an eine übergebene Telefonnummer (intern oder extern).

wol
---
Sendet einen WOL (WakeOnLAN) Befehl an eine MAC Adresse.

get_number_of_deflections
-------------------------
Liefert die Anzahl der Rufumleitungen zurück.

get_deflection
--------------
Liefert die Details der Rufumleitung der angegebenen ID zurück (Default-ID = 0)

get_deflections
---------------
Liefert die Details aller Rufumleitungen zurück.

set_deflection_enable
---------------------
Schaltet die Rufumleitung mit angegebener ID an oder aus.



Web Interface
=============

Das avm Plugin verfügt über ein Webinterface, mit dessen Hilfe die Items die das Plugin nutzen
übersichtlich dargestellt werden.

.. important::

   Das Webinterface des Plugins kann mit SmartHomeNG v1.4.2 und davor **nicht** genutzt werden.
   Es wird dann nicht geladen. Diese Einschränkung gilt nur für das Webinterface. Ansonsten gilt
   für das Plugin die in den Metadaten angegebene minimale SmartHomeNG Version.


Aufruf des Webinterfaces
------------------------

Das Plugin kann aus dem backend aufgerufen werden. Dazu auf der Seite Plugins in der entsprechenden
Zeile das Icon in der Spalte **Web Interface** anklicken.

Im WebIF stehen folgende Reiter zur Verfügung:
 - AVM Items  -  Tabellarische Auflistung aller Items, die mit dem TR-064 Protokoll ausgelesen werden
 - AVM Smarthome Items  -  Tabellarische Auflistung aller Items, die mit dem AHA Protokoll ausgelesen werden (Items der Smarthome Geräte)
 - Plugin-API  -  Beschreibung der Plugin-API
 - Log-Einträge  -  Listung der Logeinträge der Fritzbox
 - Call Monitor Items  -  Tabellarische Auflistung des Anrufmonitors (nur wenn dieser konfiguriert ist)
 - AVM Smarthome Devices  -  Auflistung der mit der Fritzbox verbundenen Geräte
