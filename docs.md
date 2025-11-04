# Nostr Client Technical Documentation: Chat and Marketplace

## 1. Project Goal and Vision

The goal of this project is to develop a **Nostr-based web client** that seamlessly integrates a secure, feature-rich **chat application** with a decentralized **peer-to-peer marketplace**. The client must prioritize user experience, security, and adherence to established Nostr Implementation Possibilities (NIPs). The documentation is structured to guide an AI coding agent through the development process.

**Inspiration:**
*   **Chat Interface & Security:** Inspired by `0xchat.com` (modern UI, secure private/group chat, voice/video call potential).
*   **Marketplace Functionality:** Inspired by `shopstr.store` and the concept of a Facebook Marketplace-style listing platform.

## 2. Core Features and Requirements

| Feature Area | Requirement | Nostr NIPs / Technical Notes |
| :--- | :--- | :--- |
| **Authentication** | Support NIP-05 (username@domain), Public Key (npub), and Private Key (nsec) login. | **NIP-05:** For username verification. The client must resolve the NIP-05 identifier to a public key. **Security:** Private key login requires a secure, temporary storage mechanism (e.g., browser extension like Alby or local encryption) and must be discouraged in favor of NIP-07 (browser extension) or NIP-46 (remote signer). |
| **Chat - Private** | Encrypted, direct messaging between two users. | **NIP-04:** Encrypted Direct Messages. Events must be Kind 4. |
| **Chat - Group** | Encrypted group chat functionality. | **NIP-24/NIP-29:** For group chat management and communication. The client should support creating and joining private, closed groups. |
| **Marketplace - Listings** | Users must be able to create, view, and manage product listings (stalls and products). | **NIP-15:** Nostr Marketplace. Events must be Kind 30017 (Stall) and Kind 30018 (Product). |
| **Marketplace - Search/Filter** | Robust search by title, description, and filtering by category and location. | Client-side indexing and filtering of NIP-15 events. |
| **Marketplace - Payments** | Integration for Lightning Network payments (SATS). | **NIP-57:** Zap (Lightning Tips) for non-product payments. For product payments, the client should support generating Lightning Invoices (LND/LNBits integration). |
| **User Interface** | Modern, responsive web design (dark mode preferred, similar to 0xchat). Clear separation between Chat and Marketplace sections. | Use a modern web framework (e.g., React, Vue, Svelte) and a component library (e.g., Tailwind CSS, Material UI). |

## 3. Technical Specifications

### 3.1. Nostr Protocol Implementation

The client must implement the following NIPs:

| NIP | Description | Implementation Details |
| :--- | :--- | :--- |
| **NIP-01** | Basic Protocol Flow | Handle `EVENT`, `REQ`, `CLOSE`, `NOTICE` messages. Connect to multiple relays. |
| **NIP-02** | Contact List and Follows | Events must be Kind 3. Used for populating the chat contact list. |
| **NIP-04** | Encrypted Direct Messages | Events must be Kind 4. Used for all private 1:1 chat. |
| **NIP-05** | Nostr Address Verification | Used for resolving `username@domain` to a public key for login and profile display. |
| **NIP-07** | Browser Extension Signer | **Recommended Login Method.** Use a browser extension (e.g., Alby) for key management and signing events without exposing the private key. |
| **NIP-15** | Nostr Marketplace | Events Kind 30017 (Stall) and Kind 30018 (Product) for marketplace listings. |
| **NIP-29** | Group Chat | Events Kind 42 (Group Chat Message) and related kinds for group creation/management. |
| **NIP-57** | Zaps (Lightning Tips) | Events Kind 9735. Used for tipping users and potentially for small-value marketplace transactions. |

### 3.2. Authentication Flow

The client must present a clear login screen with three options:

1.  **NIP-05 Login (Recommended):**
    *   User enters `username@domain`.
    *   Client resolves the NIP-05 identifier to a public key (npub).
    *   Client prompts the user to sign a dummy event or uses NIP-07 for authentication.
2.  **Public Key (npub) Login:**
    *   User enters their public key.
    *   Client prompts the user to sign a dummy event or uses NIP-07 for authentication.
3.  **Private Key (nsec) Login (Discouraged):**
    *   User enters their private key.
    *   **CRITICAL SECURITY NOTE:** The client must immediately warn the user about the security risks. The private key should be stored in an encrypted, temporary session variable and never persisted.

### 3.3. Marketplace Data Structure (NIP-15 Focus)

The marketplace will rely on NIP-15 events.

#### Stall Event (Kind 30017)

| Tag | Description | Example |
| :--- | :--- | :--- |
| `d` | Unique identifier for the stall (e.g., `my-shop-1`). | `["d", "my-shop-1"]` |
| `name` | Name of the stall. | `["name", "The Nostr Art Collective"]` |
| `description` | Description of the stall. | `["description", "Hand-drawn art and digital prints."] ` |
| `image` | URL of the stall's main image/banner. | `["image", "https://nostr.build/i/stall_banner.jpg"]` |
| `currency` | Accepted currency (e.g., SATS, USD). | `["currency", "SATS"]` |

#### Product Event (Kind 30018)

| Tag | Description | Example |
| :--- | :--- | :--- |
| `d` | Unique identifier for the product. | `["d", "product-xyz-123"]` |
| `name` | Name of the product. | `["name", "Limited Edition Nostr T-Shirt"]` |
| `description` | Detailed product description. | `["description", "High-quality cotton, size L, limited run."] ` |
| `image` | URL of the product image. | `["image", "https://nostr.build/i/tshirt_front.png"]` |
| `price` | Price of the product. | `["price", "50000"]` (50,000 SATS) |
| `currency` | Accepted currency. | `["currency", "SATS"]` |
| `p` | Reference to the Stall Event ID (Kind 30017). | `["p", "<stall_event_id>"]` |
| `t` | Category tags for filtering. | `["t", "clothing"], ["t", "t-shirt"]` |

### 3.4. User Interface Design Notes

The UI should be a single-page application (SPA) with a clear, persistent navigation structure.

| UI Component | Description | Inspiration |
| :--- | :--- | :--- |
| **Main Layout** | Two main views: **Chat** and **Marketplace**. A persistent sidebar/header for navigation. | `0xchat.com` (dark mode, clean chat list) |
| **Chat View** | Left panel: List of contacts/groups (Kind 3/NIP-29). Main panel: Chat window with message history (Kind 4/NIP-29). Input field with support for Zaps (NIP-57). | Standard modern messenger apps. |
| **Marketplace View** | **Search Bar (1):** Input for listing title, npub, or naddr. **Filters (2):** Dropdowns for Category and Location. **Listing Grid (3):** Responsive grid of product cards. | `shopstr.store` (grid layout, search/filter) |
| **Product Card** | Must display: Product Image, Title, Price (with currency), Seller's npub/NIP-05 name, and an "Active" status indicator. | `shopstr.store` product card design. |

## 4. Development Environment and Dependencies

| Component | Technology / Tool | Notes |
| :--- | :--- | :--- |
| **Frontend** | React / Next.js / SvelteKit | Choose a modern framework for a responsive SPA. |
| **Styling** | Tailwind CSS / Styled Components | For rapid, utility-first styling. |
| **Nostr Library** | `nostr-tools` (JS) or similar | For event creation, signing, and relay communication. |
| **Relay Connection** | WebSocket implementation | Must handle multiple concurrent relay connections and manage subscriptions efficiently. |
| **Key Management** | NIP-07 (Browser Extension) | Primary method for signing events. |
| **Payment** | Lightning Invoice Generation | Requires integration with a Lightning service (e.g., LNBits, LNURL-pay). |

## 5. Next Steps for AI Coding Agent

1.  **Project Setup:** Initialize the web development project (e.g., `web-static` or `web-db-user` if backend is needed for payment/user data).
2.  **Nostr Connection:** Implement the basic NIP-01 connection to a list of public relays.
3.  **Authentication:** Implement NIP-07 and NIP-05 login flows.
4.  **Chat Module:** Develop the UI and logic for NIP-04 (private chat) and NIP-29 (group chat).
5.  **Marketplace Module:** Develop the UI and logic for NIP-15 (Stall and Product events), including search and filtering.
6.  **Payment Integration:** Implement NIP-57 (Zaps) and a mechanism for generating Lightning invoices for product purchases.

---
*End of Documentation*
