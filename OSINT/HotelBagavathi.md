# L3m0nCTF – OSINT: Hotel Bagavathi

> **Category:** OSINT
> **Difficulty:** Medium
> **Flag format:** `L3m0nCTF{BSSID_TOTALBILL}`

---

## 📌 Challenge Overview

**Target:** `larry lmn`

The target leaked a link related to a new project across social media. The objective was to perform a clean OSINT pivot chain and recover **location‑bound metadata** from public artifacts.

### Objectives

* Identify the correct social media account of the target
* Pivot to the leaked project source code
* Extract hidden metadata to recover:

  * **Wi‑Fi BSSID**
  * **Total bill amount**

No brute force. No guessing. Every step had to expose the next.

---

## 🧭 High‑Level Attack Chain

The challenge followed a textbook OSINT identity‑reuse pattern:

```
VK → GitHub → Commit Logs → Spotify → Instagram → Review Platform → Pastebin → Netlify Blog
```

Each platform leaked just enough information to deterministically unlock the next.

---

## 🔍 Step‑by‑Step Breakdown

### 1️⃣ Identifying the Social Media Entry Point (VK)

**Hint:** *“It’s a famous Russian social media platform.”*

This collapses the search space immediately to **VK (VKontakte)**.

Searching for the target handle reveals the VK profile:

```
https://vk.com/larry_lmn
```

One of the posts directly links to a GitHub account — a clean identity pivot.

---

### 2️⃣ GitHub Pivot & Commit Log Analysis

From the VK profile:

```
https://github.com/larry-l3m0n
```

Relevant repository:

```
sys-monitor-v2
```

**Hint:** *“Search for keywords in GitHub commits (example: DEBUG)”*

Filtering commit messages reveals:

```
DEBUG: aHR0cHM6Ly9vcGVuLnNwb3RpZnkuY29tL3VzZXIvMzF4NnB1M3hoY2YyNm1sMzRtdWNqZDdzcWg0dT9zaT1zZGZnQWVfbVIxbUp4SkZSdXVEWDh3
```

This string is unmistakably **Base64**.

#### Decoded Output

```
https://open.spotify.com/user/31x6pu3xhcf26ml34mucjd7sqh4u?si=sdfgAe_mR1mJxJFRuuDX8w
```

Commit logs are data. Treat them that way.

---

### 3️⃣ Spotify as an Encoding Channel

Opening the Spotify profile reveals a playlist.

The encoding mechanism is subtle but intentional:

> **Take the first letter of each song title, in order.**

Extracted string:

```
instaarivudas
```

This immediately fits the structure of an Instagram handle.

---

### 4️⃣ Instagram Reconnaissance

Visiting:

```
https://instagram.com/instaarivudas
```

Reels on the account contain screenshots of chat conversations. From these:

* Location mentioned: **Srinagar**
* Venue type: **Café**
* Complaints:

  * Poor Wi‑Fi
  * Less sugar in coffee
* Explicit mention of leaving a **1‑star review**

This provides both **geographic** and **platform** pivots.

---

### 5️⃣ Café Identification & Review Correlation

The chats reference:

* Movie: **Leo**
* Café name: **Sifar**

Searching review platforms reveals **Sifar Café** in the Srinagar region on **TripAdvisor**.

A 1‑star review by user `88fFyTM1` contains the payload:

```
"...the wifi was very poor and the ssid of the wifi is 00:1A:2B:3E:4D:5A"
```

Despite saying *SSID*, the value is clearly a **BSSID (MAC address)**.

✅ **Recovered BSSID:**

```
00:1A:2B:3E:4D:5A
```

---

### 6️⃣ Pastebin Pivot via Identifier Reuse

Reviewer ID:

```
88fFyTM1
```

This matches the exact structure of a Pastebin paste ID.

Constructed URL:

```
https://pastebin.com/88fFyTM1
```

Contents:

```
https://l3m0n-vetrivel.netlify.app/
```

Classic OPSEC failure: reusing usernames as storage identifiers.

---

### 7️⃣ Netlify Blog & Bill Extraction

The Netlify site hosts a stylized logbook.

Entry of interest:

```
SRINAGAR F4 STREET DEMO
```

This entry contains an itemized bill:

* Black Coffee: ₹96.87
* Chicken Kebab: ₹387.85
* Tip: ₹10.00

**Total:**

```
₹494.72
```

---

## 🏁 Final Flag

```
L3m0nCTF{00:1A:2B:3E:4D:5A_494.72}
```

---


**OSINT isn’t magic. It’s patience plus pattern recognition.**
