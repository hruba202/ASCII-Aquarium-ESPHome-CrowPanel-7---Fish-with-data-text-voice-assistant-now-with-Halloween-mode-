# ASCII Aquarium pro ESPHome (CrowPanel 7")

Externí komponenta ESPHome, která na displeji vykresluje ASCII akvárium
s plavajícími rybami, bublinami, mořským dnem a hodinami. Vznikla na CYD
(ESP32, 320x240) a je přenesená na **Elecrow CrowPanel 7" (ESP32-S3, 800x480)**
s hlasovým asistentem.

## Funkce
- Ryby plavou vpřed i nahoru/dolů, každá má náhodnou barvu
- Teplotní ryba s živou hodnotou ze senzoru
- Ryba "POWEREDbyHOMEASSISTANT"
- Krmení: padající vločky, za kterými ryby plavou (spouští se z HA)
- Záhlaví s malými hodinami a datem, velké ASCII hodiny, hodiny v rámečku
  s průhledným pozadím (každé lze zapnout/vypnout přepínačem v HA)
- Nastavitelný počet ryb a rychlost
- **Halloween režim:** místo ryb plavou ikony MDI (dýně, duchové, lebky,
  pavouci, pavučina)
- **Postrašení:** blikající obrázek ducha/lebky na displeji a strašidelný zvuk
  spuštěný automatizací v Home Assistantu

## Struktura
```
my_components/ascii_aquarium/
  __init__.py        # schéma konfigurace (width, height, char_width, line_height)
  ascii_aquarium.h   # logika, fyzika ryb, kreslení hodin
crowpanel-aquarium.yaml
```

## Konfigurace (výřez)
```yaml
external_components:
  - source:
      type: local
      path: my_components

ascii_aquarium:
  id: my_aquarium
  width: 800
  height: 480
  char_width: 9.6
  line_height: 16
```
V display lambdě získáš komponentu přes `auto* aq = id(my_aquarium);`.
Teplotu předáš voláním `aq->set_temperature(x)`.

## Ikony a obrázky
- Font `mdi_font` (Material Design Icons, **velikost max. 16** – větší
  na tomto displeji padá) se používá pro Halloween ikony.
- Strašidelný obrázek: komponenta `image` (`mdi:ghost`, `mdi:skull`),
  přepínaná skriptem `scare_script`.

## Známé problémy
- Při přehrávání zvuku může během strašidelného obrázku zadrhávat reproduktor
  (`ISR event queue overflow`) – pomáhá zvýšit `update_interval` displeje
  (např. 40 ms).
- Kód nebyl automaticky testován, ověřeno jen na vlastním CrowPanelu.

## Hlasový asistent
Panel funguje zároveň jako hlasový satelit pro Home Assistant:
- Probouzecí slovo (`micro_wake_word`) a `voice_assistant` komunikují s HA
- Mikrofon: I2S (PDM), reproduktor: I2S, přehrávání zvuků přes `media_player` (`speaker`)
- Na displeji se pod záhlavím ukazuje stav asistenta:
  `LISTENING` → `THINKING` → `SPEAKING` (podle proměnné `va_state`)
- Reproduktor sdílí asistent i strašidelné zvuky z Home Assistantu
  (`media_player.play_media` s `announce: true`)
- Podsvícení a zesilovač se ovládají přes I2C (adresa 0x30), dotyk GT911 (0x5D)

> Piny a nastavení audia jsem převzal z vlastních funkčních konfigurací
> (`asistent_test`, `asistent1`). Pokud používáš jiný hardware, uprav je.

# ASCII Aquarium for ESPHome (CrowPanel 7")

An ESPHome external component that renders an ASCII aquarium on a display:
swimming fish, bubbles, a seabed and clocks. It started on a CYD
(ESP32, 320x240) and was ported to the **Elecrow CrowPanel 7" (ESP32-S3, 800x480)**
with a voice assistant.

## Features
- Fish swim forward and also up/down, each with a random color
- A temperature fish showing a live value from a sensor
- A "POWEREDbyHOMEASSISTANT" fish
- Feeding: falling flakes that the fish chase (triggered from Home Assistant)
- Header with a small clock and date, a big ASCII clock, and a boxed clock
  with a transparent background (each can be toggled with a switch in HA)
- Adjustable number of fish and speed
- **Halloween mode:** MDI icons swim instead of fish (pumpkins, ghosts,
  skulls, spiders, spider web)
- **Scare effect:** a blinking ghost/skull image on the display and a spooky
  sound played by a Home Assistant automation

## Voice assistant
The panel doubles as a voice satellite for Home Assistant:
- Wake word (`micro_wake_word`) and `voice_assistant` talk to HA
- Microphone: I2S (PDM), speaker: I2S, sounds are played through a `speaker` `media_player`
- The assistant state is shown under the header on the display:
  `LISTENING` → `THINKING` → `SPEAKING` (driven by the `va_state` variable)
- The speaker is shared by the assistant and the spooky sounds from Home Assistant
  (`media_player.play_media` with `announce: true`)
- Backlight and amplifier are controlled over I2C (address 0x30), touch is a GT911 (0x5D)

> Pins and audio settings were taken from my own working configs
> (`asistent_test`, `asistent1`). Adjust them if you use different hardware.

## Structure
```
my_components/ascii_aquarium/
  __init__.py        # config schema (width, height, char_width, line_height)
  ascii_aquarium.h   # logic, fish physics, clock drawing
crowpanel-aquarium.yaml
```

## Configuration (excerpt)
```yaml
external_components:
  - source:
      type: local
      path: my_components

ascii_aquarium:
  id: my_aquarium
  width: 800
  height: 480
  char_width: 9.6
  line_height: 16
```
In the display lambda, get the component with `auto* aq = id(my_aquarium);`.
Pass the temperature with `aq->set_temperature(x)`.

## Icons and images
- The `mdi_font` font (Material Design Icons, **max. size 16** – larger
  sizes crash this display) is used for the Halloween icons.
- Scare image: an `image` component (`mdi:ghost`, `mdi:skull`),
  toggled by the `scare_script` script.

## Known issues
- When a sound plays while the scare image is shown, the speaker may stutter
  (`ISR event queue overflow`) – increasing the display `update_interval`
  (e.g. to 40 ms) helps.
- The code is not covered by automated tests; it was only verified on my own CrowPanel.
