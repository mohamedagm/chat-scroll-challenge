# 💬 Chat Auto-Scroll Challenge

A Flutter chat app powered by the Gemini API, with a focus on delivering a smooth and intelligent auto-scroll experience during AI response streaming.

---

## 🌐 Deployed URL

**[Live Demo](#)** 

---

## 🛠️ UX Issues Identified & Fixed

The original app had no scroll logic whatsoever — messages streamed in but the list never moved. Below is a breakdown of every issue found and how it was resolved.

---

### 1. `_isAutoScrollEnabled` Flag

**Problem:** No mechanism existed to track whether the user had intentionally scrolled away or was still following the stream.

**Fix:** Added a single boolean flag `_isAutoScrollEnabled` as the source of truth. All scroll decisions read from or write to this flag, keeping the logic centralized and predictable.

---

### 2. Unified `_scrollToBottom({force})` Method

**Problem:** Scroll calls were scattered and inconsistent.

**Fix:** Centralized all scrolling into one function with a `force` parameter:
- `force: true` → uses `jumpTo()` for an instant snap _(used when sending a new message)_
- `force: false` → uses `animateTo()` for smooth scrolling _(used during streaming)_

---

### 3. Streaming Updates Connected to Scroll

**Problem:** The list never scrolled as new chunks arrived from the AI.

**Fix:** Added `_streamManager.addListener(_onStreamUpdate)` so that every new streaming chunk triggers a scroll — but only when streaming is active **and** auto-scroll is enabled.

---

### 4. `NotificationListener<ScrollNotification>` Wrapper

**Problem:** There was no way to distinguish between a user scrolling manually and the app scrolling programmatically.

**Fix:** Wrapped the chat list with a `NotificationListener<ScrollNotification>` to intercept all scroll events and determine their origin before deciding whether to pause or resume auto-scroll.

---

### 5. Scroll Policy in `_onScrollNotification`

**Problem:** Auto-scroll had no awareness of user intent.

**Fix:** Built a scroll policy that:
- Uses `dragDetails != null` to detect **real user gestures only** (not programmatic scrolls)
- If the user scrolls **up** → disables auto-scroll so they can read freely
- If the user returns to the **bottom** (by drag or momentum) → re-enables auto-scroll

---

### 6. Correct Behavior When Sending a New Message

**Problem:** Sending a message while scrolled up left the user stranded — the new response started streaming off-screen.

**Fix:** In `_handleMessageSend`, before starting the new stream:
1. Resets `_isAutoScrollEnabled = true`
2. Forces an instant scroll snap to the bottom via `_scrollToBottom(force: true)`
3. Then starts the streaming response

---

### 7. Flick / Momentum Scroll Handled Correctly

**Problem:** Fast flick gestures that land at the bottom weren't detected, so auto-scroll stayed paused even after the user reached the bottom naturally.

**Fix:** Added handling for `ScrollEndNotification` — after any momentum scroll ends, the position is checked and auto-scroll is re-enabled if the user landed near the bottom.

---

## 🎬 Screen Recordings

| Scenario | Recording |
|----------|-----------|
| Scenario 1 — Basic Auto-Scroll | [Watch Recording](#) |
| Scenario 2 — Pause on Manual Scroll Up | [Watch Recording](#) |
| Scenario 3 — Send While Scrolled Up | [Watch Recording](#) |
| Scenario 4 — Resume Auto-Scroll on Scroll Down | [Watch Recording](#) |

---