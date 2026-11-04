# Bluetti2MQTT Plus — Home Assistant Add-ons

This repository contains **two separately installable add-ons**:

| Add-on | Version | Supported use case |
|---|---|---|
| **Bluetti2MQTT Plus** | **3.0.3 (legacy)** | Existing unencrypted Bluetooth devices and the previously supported Bluetti model range. Unchanged. |
| **Bluetti2MQTT Plus - AC500 Secure BLE** | **4.0.0** | Standalone bridge for **Bluetti AC500 with encrypted BLE** after firmware updates. |

The new secure BLE add-on lives under [`bluetti2mqtt_ac500_ble/`](bluetti2mqtt_ac500_ble/README.md); legacy remains under [`bluetti2mqtt/`](bluetti2mqtt/).

**Important:** AC500 Secure BLE 4.0.0 is **not** a drop-in replacement for every model supported by the legacy add-on. Do **not** install both add-ons for the same AC500 MQTT device identity. Backup your current add-on configuration before switching.

The v4 runtime does not depend on semitop7: it uses `bluetti-bt-lib==0.1.8` and the standalone MQTT bridge. Modbus hybrid support is planned separately and is **not included** in v4.0.0.

See [v4.0.0 release notes](RELEASE_NOTES_v4.0.0.md).

---

## Legacy 3.0.3 documentation (preserved)

# Bluetti2MQTT_Plus

Fork di [semitop7/bluetti2mqtt](https://github.com/semitop7/bluetti2mqtt) con aggiunta del supporto per `grid_charge_amps`, `DC Input Power 2` e `DC Input Voltage 2`.

## Cosa fa

Questo addon espone i dati della Bluetti tramite MQTT e Home Assistant, e aggiunge il controllo degli ampere di carica da rete (`grid_charge_amps`)  

Supporto ac500 ac300 ac200L ep500 ep500p ep600 (testato su ac500)

Su **AC500** implementati i campi **DC Input Power 2 e DC Input Voltage 2**

## Novità rispetto a semitop7

- ✅ Aggiunto il campo `grid_charge_amps` ai modelli su elencati
- ✅ Configurato come entità leggibile e scrivibile in Home Assistant
- ✅ Consente di controllare la carica da rete in tempo reale senza riavviare l'addon
- ✅ Aggiunti i campi `DC Input Power 2 e DC Input Voltage 2` su AC500 (replicabile su altri modelli)

## Installazione

1. **In Home Assistant**, vai a: **Impostazioni → Sistemi → Repository**
2. **Clicca "+"** e aggiungi:

https://github.com/LinusIta/bluetti2mqtt_plus

3. **Vai in Componenti Aggiuntivi** e cerca **"Bluetti2MQTT Plus"**
4. **Clicca Installa**
5. **Configura:**
   - MQTT Host: indirizzo del tuo broker MQTT
   - MQTT Username/Password: credenziali
   - BT MAC: indirizzo Bluetooth della tua Bluetti
   - Poll interval: secondi tra i polling (default 30)
6. **Avvia l'addon**

## Utilizzo

### Leggere i dati

Tutti i dati della Bluetti sono disponibili come entità MQTT in Home Assistant compreso:

- `number.XXXXX_XXXX_grid_charge_amps` 

### Impostare gli ampere di carica

**Via MQTT Home Assistant:**
action: mqtt.publish
data:
  topic: bluetti/command/XXXX-XXXXXXXXX/grid_charge_amps
  payload: "4"

(inserire nome corretto del topic mqtt)

**Via UI Home Assistant:**
1. Vai in **Sviluppo → Stati**
2. Seleziona l'entità `number.XXXXX_XXXX_grid_charge_amps`
3. Modifica il valore e imposta stato


## Configurazione MQTT avanzata

L'addon supporta i seguenti modi:
- `mqtt`: Pubblica su MQTT (default)
- `discovery`: Discovery automatico Home Assistant
- `logger`: Solo log

Vedi la configurazione nell'addon per maggiori dettagli.

## Problemi comuni

**L'addon non si connette al Bluetooth:**
- Verifica che l'indirizzo MAC sia corretto
- Assicurati che il Bluetti sia acceso e vicino
- Riavvia l'addon

**grid_charge_amps non appare:**
- Disinstalla e reinstalla l'addon (forza il rebuild)
- Controlla i log: **Impostazioni → Componenti Aggiuntivi → Bluetti2MQTT Grid → Log**

## Crediti

- [semitop7](https://github.com/semitop7) per il lavoro originale su bluetti2mqtt
- [warhammerkid](https://github.com/warhammerkid) per il supporto Bluetooth
- Questa fork aggiunge il supporto per `grid_charge_amps`, `DC Input Power 2` e `DC Input Voltage 2`

## License

MIT - Vedi LICENSE file
