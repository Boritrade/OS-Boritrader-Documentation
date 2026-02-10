# **The Boritrader Documentation: Patch Notes August 4 to August 19, 2025**
**Boritrade, LLC**   
**Related Release Version:** 0.9.4   


**Focus:** notification UX — **audible alerts**, **signal-change highlighting**, and **user sound preferences**; plus minor JS fixes and settings UX polish.

---

## **Overview**

* **Audible alerts** added with a sliding-window **rate limit (max 5 plays / 8 min)** and browser **autoplay-unlock** via a single shared audio element.
* **Signal-change detection** surfaces transitions (e.g., **HOLD → BUY**) directly in the UI.
* **User sound preferences**: selectable from a **whitelisted set (boritrade-1..7.mp3)** with a **“Test sound”** action.
* **Settings UX**: each section now saves independently; reduces accidental cross-form changes.
* **JS fixes/cleanup**: algorithm name pass-through corrected; removed defunct parsing.

---

## **Key Updates**

### **Audible Alerts**

* Plays a notification sound when a **new signal is received**; updated to play **only on signal change** based on later commit.
* **Rate limiting:** sliding window of **5 plays per 8 minutes** to prevent alert fatigue.
* **Autoplay unlock:** one shared audio player satisfies modern browser policies; activated after first user interaction.

### **Signal-Change Highlighting**

* UI now explicitly displays **status transitions** (e.g., **BTCUSDT: HOLD → BUY**) with an arrow indicator.
* Pairs with centralized runner + standardized terms (**BUY/SELL/HOLD**).

### **Sound Preferences in Settings**

* Users can choose from **approved audio files** (`boritrade-1.mp3` … `boritrade-7.mp3`).
* Server resolves a **safe static URL**; preferences are embedded via `json_script`.
* **“Test sound”** button to preview selection.

### **Settings Form UX**

* **Per-section Save**: removed single global submit; each card/section saves independently to reduce friction and mistakes.

### **Bug Fixes & Cleanup**

* Fixed: **algorithm name** not passed correctly to client logic.
* Removed **defunct `parseMessage`** helper to prevent stale variable handling.
* Kept production JS setup intact; minor dev-ready adjustments.


