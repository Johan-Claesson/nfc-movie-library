# NFC Movie Library for Plex (Home Assistant + ESP32)

An NFC-based movie library that lets kids start their favorite movies or TV shows by simply tapping a card.

This project was inspired by [u/Xafke's](https://www.reddit.com/user/Xafke/) NFC movie library. I built my own version using **ESP32**, **Home Assistant**, and **Plex** so my 2-year-old can start movies independently.

The idea is simple:  
tap an NFC card → Home Assistant receives the tag → a Automation calls the Plex API → playback starts on the Apple TV.
Original Reddit post: https://www.reddit.com/r/homeassistant/comments/17xdxti/how_i_built_an_nfc_movie_library_for_my_kids_ha/ 

![NFC-Scanner and cards](https://github.com/Johan-Claesson/nfc-movie-library/blob/main/images/IMG_1334.JPEG)

---

# How it works

The system flow looks like this:
```
NFC Tag
↓
ESP32 + PN532
↓
Home Assistant
↓
Home Assistant Automation
↓
Plex API
↓
Apple TV / Plex Player
```

When a tag is scanned:

1. The **ESP32 reads the NFC tag**
2. The tag ID is sent to **Home Assistant**
3. A **Home Assistant Automation** is triggered
4. The Automation calls the **Plex API**
5. Plex starts playback on the selected player

---

# Hardware

All components were purchased from **AliExpress** and the total cost was under **100 SEK** about 10 USD or EUR.

Required hardware:

- **ESP32-C3 Mini**
- **PN532 NFC Reader**
- **NFC tags/cards**
- **USB-C cable**
- enclosure / box 52 x 82 x 35 mm
- Jumper cables, and soldering stuff. 

I originally tried a **3D printed case**, but it was too shallow for the USB-C cable.  
I'm currently testing a plastic enclosure sized: 52 x 82 x 35 mm 


Minimum internal height required is about **25 mm**.

---

# Home Assistant Setup

This project uses:

- **ESPHome** for the ESP32
- **Plex Integration** in Home Assistant
- **Home Assistant Automations**
- **Plex API calls**

Important notes: 

I initally only used external services such as TV4Play, Disney, SVTPlay, Max, but there is a lot of things that you cant control in some of those apps, like choosing who is watching - which was a game stopper for me. Also, we hade a weekend without internet access, so a local-first approach was important for us, and therefore Plex was the best choice. 

## Stop playback before starting a new video

Sometimes the Plex app on Apple TV would freeze when starting playback.  
To fix this, the Automation first sends a **stop command** to the media player before starting a new item.
```YAML
  - action: media_player.media_stop
    metadata: {}
    target:
      entity_id: media_player.vardagsrum
```

## Force playback from the beginning

Plex normally resumes playback.  
To avoid this, the Automation marks the item as **watched before playback starts**, ensuring it always begins from the start.
Not sure why it says token: "[object Object]": null in the code, but it works. 
```YAML
action: rest_command.plex_mark_watched
metadata: {}
data:
  rating_key: "{{ nfc_mapping[trigger.event.data.tag_id].plex_id }}"
  token:
    "[object Object]": null
```

## Disable intro and credits detection (recommended for kids content)

Short kids' shows can lose a lot of content when auto-skip is enabled.

Disable this in Plex:

1. Edit the library
2. Scroll near the bottom
3. Disable:
   - `Enable intro detection`
   - `Enable credits detection`

---

# Plex Collections

To make content easy to trigger, this setup uses **Smart Collections** in Plex. Xafke covered this in his guide: https://simplyexplained.com/blog/how-i-built-an-nfc-movie-library-for-my-kids/ 

When an NFC tag is scanned:

1. Home Assistant queries the **Plex API**
2. The Automation retrieves the **first item in the collection**
3. That item is played on the selected device

This allows one card to represent:

- a movie
- a TV show
- a playlist
- a themed collection

---

# NFC Cards

Each NFC card represents a **movie, show, or collection**.

Examples:

- Frozen
- Paw Patrol
- Bluey
- Disney Movies

The NFC tag UID is mapped in the **Home Assistant automation**.

---

# Project Goals

The goal of this project was to:

- make media accessible for a **2-year-old**
- remove the need to navigate Plex menus
- create a **physical media library**
- make it simple and cheap to build

---

# Credits

Huge credit to:

[u/Xafke](https://www.reddit.com/user/Xafke/)  for the original idea and inspiration. 
Here is the post that he wrote: https://simplyexplained.com/blog/how-i-built-an-nfc-movie-library-for-my-kids/ 

---

If you're interested in building something similar, feel free to use the configs in this repo.
