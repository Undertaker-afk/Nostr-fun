# Nostr Client Integration Plan: Web App and Android Sync Blueprint

## 1. Introduction and Goal

This document outlines the **Integration Plan** for the Nostr Chat and Marketplace client. The primary goal is to establish a robust, cross-platform architecture that supports the immediate development of a feature-rich **Web Application** while laying the groundwork for seamless data synchronization with a future **Android Application**.

The core principle of this architecture is **"Relay-Centric Data"**, leveraging the Nostr protocol's inherent decentralized nature to manage state and synchronization, minimizing the need for a custom backend server.

## 2. Web Application Architecture (Phase 1: Immediate Focus)

The Web Application will be a **Single Page Application (SPA)** built with a modern JavaScript framework (e.g., React/Next.js, SvelteKit) to ensure a responsive and fast user experience, as inspired by `0xchat.com`.

### 2.1. Technology Stack

| Component | Technology / Tool | Rationale |
| :--- | :--- | :--- |
| **Frontend Framework** | React / Next.js (Recommended) | Excellent ecosystem, server-side rendering (SSR) for performance, and strong community support. |
| **Styling** | Tailwind CSS | Utility-first approach for rapid, consistent UI development. |
| **Nostr Library** | `nostr-tools` (JS) or similar | Essential for event creation, signing, and relay communication. |
| **State Management** | Zustand / Redux Toolkit | Efficiently manage client-side state, especially for real-time data from relays. |
| **Key Management** | NIP-07 / NIP-46 | Secure, non-custodial key handling. |

### 2.2. Data Flow and State Management

1.  **Event Subscription:** The client connects to a curated list of relays (NIP-01) and subscribes to relevant events based on the user's public key (e.g., Kind 4 for DMs, Kind 30017/30018 for Marketplace listings).
2.  **Local Cache:** All received events are stored in a fast, client-side database (e.g., IndexedDB or a state management layer like Redux/Zustand). This cache is crucial for a smooth user experience, allowing for quick retrieval of past messages and listings without constant relay requests.
3.  **Real-time Updates:** The client maintains persistent WebSocket connections to relays to receive new events in real-time, updating the local cache and UI instantly.

## 3. Android Synchronization Blueprint (Phase 2: Future Integration)

The Android application will be built using a native framework (e.g., Kotlin/Jetpack Compose) or a cross-platform solution (e.g., React Native/Flutter) that can efficiently handle WebSocket connections and local data storage.

### 3.1. Synchronization Strategy: Relay-Centric

Synchronization between the Web App and the Android App is **natively handled by the Nostr protocol and the relays**. Both clients will connect to the same set of relays and subscribe to the same events.

| Synchronization Aspect | Web App Implementation | Android App Implementation |
| :--- | :--- | :--- |
| **Core Data** | Events (Kind 1, 4, 30017, etc.) | Events (Kind 1, 4, 30017, etc.) |
| **Relay List** | Stored in local storage (Kind 10002 for relay list management). | Stored in local database (Kind 10002 for relay list management). |
| **Local Storage** | IndexedDB / Local Storage | SQLite / Room Database |
| **State Consistency** | Handled by the relay's event-based, append-only nature. The latest event (highest `created_at` timestamp) is the source of truth. | Handled by the relay's event-based, append-only nature. The latest event (highest `created_at` timestamp) is the source of truth. |

### 3.2. Key Synchronization Challenges and Solutions

| Challenge | Description | Solution (Nostr-Native) |
| :--- | :--- | :--- |
| **Read Receipts** | Ensuring both clients know which messages have been read. | Implement a custom event kind (e.g., Kind 15, following NIP-58 for custom kinds) for "Read Receipt" events, published to relays by the client when a message is viewed. |
| **Drafts/Local State** | Data that should *not* be public (e.g., a message draft, UI preferences). | **DO NOT** publish to relays. Store locally on each device (IndexedDB for Web, Room for Android). |
| **Initial Load Time** | Fetching all history on a new device can be slow. | Implement **NIP-40 (Expiration Tag)** for ephemeral data and optimize subscription filters to only fetch recent events first, then backfill history in the background. |

## 4. Integration Details for AI Coding Agent

The AI Coding Agent should prioritize the following architectural decisions to ensure future Android compatibility:

### 4.1. Authentication and Key Management

The most critical component for cross-platform sync is the secure, shared key.

*   **Primary Method:** **Nostr Connect (NIP-46)**. This is the ideal solution as the private key is never exposed to either the Web or Android client. Both clients simply connect to the same remote signer (e.g., Alby Mobile) using their respective `nostrconnect://` URIs, ensuring a single source of truth for signing events.
*   **Secondary Method:** **NIP-07** for the Web App only.
*   **NIP-05:** Used for resolving the human-readable name to the public key, which is consistent across all platforms.

### 4.2. Marketplace Integration (NIP-15)

The marketplace events (Kind 30017/30018) are inherently public and decentralized.

*   **Web App:** Should be able to publish new Stall and Product events, and handle payment requests via NIP-47.
*   **Android App:** Will subscribe to the same events, ensuring that a listing created on the Web App is immediately visible on the Android App (and vice-versa) once the relay propagates the event, and handle payment requests via NIP-47.

### 4.3. Chat Integration (NIP-04/NIP-29) and Payments

*   **Encryption:** All private chat events (Kind 4) must use the standard NIP-04 encryption. Since the encryption/decryption is based on the shared private key, and the private key is managed by the NIP-46 remote signer, both the Web and Android clients will be able to seamlessly encrypt and decrypt messages.
*   **Group Chat:** NIP-29 events must be consistently implemented on both platforms to ensure group membership and message history are synchronized.
*   **Payments in Chat:** Implement NIP-57 (Zaps) and NIP-47 for sending/receiving SATS directly within the chat interface.

## 5. Lightning Wallet Integration (NIP-47 / LNURL)

To enable the "chat, shop, and manage funds" vision, **Nostr Wallet Connect (NIP-47)** is the critical integration point.

### 5.1. NIP-47 (Nostr Wallet Connect)

NIP-47 allows the client to securely request payments from a user's remote Lightning wallet without ever holding the user's funds or keys.

| Feature | NIP-47 Implementation |
| :--- | :--- |
| **Sending Payments** | Client sends a `pay_invoice` request to the user's NWC URI. |
| **Receiving Payments** | Client generates a Lightning Invoice and sends a `get_invoice` request to the NWC URI (if the wallet supports this). More commonly, the client uses a Lightning Address (LNURL-pay) associated with the user's wallet. |
| **Non-Custodial** | The user's wallet remains separate and secure, connected only via the NWC string. |

### 5.2. LNURL Integration

LNURL (Lightning URL) simplifies payment requests and withdrawals.

*   **LNURL-pay:** Used for generating Lightning Invoices for marketplace purchases and for receiving Zaps (NIP-57).
*   **LNURL-withdraw:** Used for users to withdraw funds from any custodial balance (if the client ever implements one, though non-custodial is preferred) or for automated payouts.

## 6. Conclusion

By strictly adhering to the Nostr protocol and prioritizing NIP-46 for key management, the Web Application will be built with a **decentralized, mobile-ready foundation**. The data synchronization challenge is effectively outsourced to the robust, event-based nature of the Nostr relays, ensuring a consistent user experience across the Web and future Android applications.

---
*End of Integration Plan*
