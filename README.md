# Zendesk Messenger (SDK) Chatbot Tracking for GTM 💬📊

This repository contains a solution for tracking user interactions with the **Zendesk Messaging Widget** (newer version) using **Google Tag Manager (GTM)**.

Since the Zendesk widget loads within an **iframe**, standard GTM Click or Element Visibility triggers cannot detect interactions. This solution bridges the gap between the Zendesk widget and your analytics platform (like GA4) by using the **Zendesk JavaScript API (`zE`)** to "listen" for widget events and push them to the Data Layer.

## 🚀 Key Features

The script uses a robust "polling" mechanism to ensure the Zendesk `zE` object is fully loaded before attaching listeners. It tracks:

* **Chat Opened:** Fires when a user clicks to open the messenger.
* **Chat Closed:** Fires when a user minimizes the messenger.
* **Reply Received:** Fires when the bot or an agent sends a message, including a count of unread messages.
* **Performance Safe:** Stops checking after 20 seconds if the widget is not found (e.g., blocked by ad blockers) to save browser resources.
* **Clean Data:** Automatically handles Data Layer persistence by resetting unread counts to `undefined` on open/close events to prevent reporting errors.

## 💡 Why this is useful

1.  **User Engagement Insights:** Understand what percentage of your web traffic actually interacts with your support bot.
2.  **Conversion Attribution:** See if interacting with the chatbot correlates with higher conversion rates or lower bounce rates.
3.  **Unread Message Awareness:** Track if users are leaving the site while still having unread replies from your team.

## 🛠 Prerequisites

* **Google Tag Manager** installed on your website.
* **Zendesk Messaging Widget** (The newer "Messenger" version, not the Classic Web Widget).
* **Google Analytics 4 (GA4)** configuration tag set up in GTM.

## 📦 Implementation Guide

### Step 1: Create the Listener Tag

1.  Download or copy the script file included in this repository (e.g., `script.html`).
2.  In GTM, create a new **Custom HTML Tag**.
3.  Name it: `Script - Zendesk Listener`.
4.  Paste the code from the file into the HTML field.
5.  **Trigger:** Fire on **All Pages** (or pages where the Chatbot is present).

---

### Step 2: Create Data Layer Variables

Create two **Data Layer Variables** in GTM to capture the data pushed by the script.

| Variable Name in GTM | Data Layer Variable Name | Description |
| :--- | :--- | :--- |
| `dlv - chatbot_status` | `chatbot_status` | Captures 'open', 'close', or 'reply_received' |
| `dlv - chatbot_unread_messages` | `chatbot_unread_messages` | Captures the number of unread messages |

---

### Step 3: Create the Trigger

Create a **Custom Event Trigger** to tell GTM when to send the data to GA4.

* **Trigger Type:** Custom Event
* **Event Name:** `chatbot_interaction`
* **Trigger Name:** `Custom Event - Chatbot Interaction`

---

### Step 4: Configure the GA4 Event Tag

Create a single **GA4 Event Tag** to handle all interactions.

* **Tag Type:** Google Analytics: GA4 Event
* **Event Name:** `chat_interaction`
* **Event Parameters:**

| Parameter Name | Value |
| :--- | :--- |
| `interaction_status` | `{{dlv - chatbot_status}}` |
| `unread_count` | `{{dlv - chatbot_unread_messages}}` |

* **Trigger:** `Custom Event - Chatbot Interaction`

## 🧠 Technical Details & Logic

### Why `undefined`?
GTM's Data Layer has a "persistence" feature where previous values stay in memory until overwritten.
* **Problem:** If a user receives a reply (`count: 1`), GTM remembers "1". If they later open the chat (where count is irrelevant), GTM would attach that old "1" to the Open event, polluting your data.
* **Solution:** The script explicitly sends `undefined` when the status is `open` or `close`. This forces GA4 to drop the `unread_count` parameter entirely for those events, keeping your reports clean.

### Safety Timeout
The script uses a `setInterval` loop to check for the Zendesk `zE` object every 1 second. It includes a `maxAttempts` counter (set to 20). If Zendesk fails to load (e.g., due to an AdBlocker or network error), the script stops running to preserve browser resources.

## 🧪 How to Verify

1.  Open GTM **Preview Mode**.
2.  Go to your website and open the Chatbot.
3.  In the GTM Debugger, look for the `chatbot_interaction` event.
4.  Check the **API Call** tab to ensure:
    * `chatbot_status`: 'open'
    * `chatbot_unread_messages`: `undefined` (Correct behavior).
5.  Minimize the chat and wait for a reply (or trigger one).
6.  Check the new event:
    * `chatbot_status`: 'reply_received'
    * `chatbot_unread_messages`: `1`
