# Delivery_Docket[README.md.md](https://github.com/user-attachments/files/24050473/README.md.md)
# Daily Docket – Technical Specification & Architecture Document

**Platform:** Native Android  
**Language:** Kotlin  
**UI Toolkit:** Jetpack Compose (Material 3)  
**Architecture:** MVVM + Clean Architecture  
**Core Philosophy:** Offline-First • High-Speed • Business-Critical  
**Theme:** Monokai Dimmed (Dark Mode Only)

---

## 1. Project Overview

Daily Docket is a native Android application designed for delivery teams who must operate efficiently in the field without internet access.

The app replaces a legacy web-based invoice/OCR workflow with a fast, offline-first, camera-driven **"Digital Clipboard"** that allows drivers to:

- Capture invoices using OCR
- Confirm delivery details
- Capture POD photos
- Obtain customer signatures
- Reorder their route dynamically
- Export daily delivery logs (CSV/JSON)

All data is stored directly on the device using **Room**.

---

## 2. Business Objectives → Technical Requirements

| Business Objective | Technical Implementation |
|-------------------|-------------------------|
| Work Anywhere (Offline) | Room Database as the single source of truth. No external servers. |
| Fast OCR (No Network) | Google ML Kit Text Recognition v2 — fully on-device, real-time. |
| Zero-Friction Workflow | Jetpack Compose, custom CameraX interface, no file pickers. |
| High Data Density | "Smart List" route screen (not a spreadsheet), drag-to-reorder. |
| Accurate Delivery Verification | Invoice image → items → signature pad placed directly below. |
| Reliable Proof Collection | POD image capture stored locally. |
| Exportable Business Intelligence | CSV/JSON export via FileProvider. |
| Ergonomic Visual Design | "Monokai Dimmed" dark theme with elevation-based depth. |

---

## 3. Technology Stack (MAD-Compliant)

### 3A. Language & Core Libraries

| Component | Reason |
|-----------|--------|
| Kotlin | Null safety, coroutines, modern Android standard. |
| Coroutines + Flow | Smooth background operations for OCR & DB. |
| Hilt | Dependency injection to avoid manual wiring. |
| Gson | JSON-encoding appliance lists. |

### 3B. Capture & Recognition ("The Eyes")

| Tool | Purpose |
|------|---------|
| CameraX | Embedded camera preview & capture. |
| ML Kit Text Recognition v2 | Fast local OCR on invoices. |
| Coil | Efficient image loading (POD + signature PNG). |

### 3C. Interface Layer ("The Face")

| Tool | Purpose |
|------|---------|
| Jetpack Compose (Material 3) | Declarative UI system for Android. |
| Navigation Compose | Handles screen-to-screen flow. |
| Material Icons | Maps, Phone, Camera icons. |

---

## 4. Theme & Branding – Monokai Dimmed (Dark Mode Only)

Daily Docket uses a **Monokai Dimmed**–inspired color palette for high contrast in vehicles under varied lighting.

| Role | Hex |
|------|-----|
| Background | `#191919` |
| Surface | `#2D2A2E` |
| Primary (Action Green) | `#A6E22E` |
| Secondary (Cyan Accents) | `#66D9EF` |
| Tertiary (Alert Pink) | `#F92672` |
| Text (OnSurface) | `#F8F8F2` |

### Elevation Rules

- **Cards:** 2–4dp
- **FABs:** 6–8dp with shadow
- **Signature & POD blocks:** tonalElevation 2dp

---

## 5. Data Architecture – Room Schema

### InvoiceRecord Entity

This is the **single source of truth**.

| Field | Type | Description |
|-------|------|-------------|
| `id` | Long (PK) | Auto-generated ID |
| `invoiceNumber` | String | Extracted via OCR regex |
| `customerName` | String | Extracted |
| `customerAddress` | String | Extracted |
| `customerPhone` | String | Extracted |
| `items` | String | JSON list of appliances/services |
| `status` | String | "Open", "Delivered", etc. |
| `podImagePath` | String? | Saved URI to POD image |
| `signaturePath` | String? | Saved URI to signature PNG |
| `driverNotes` | String | Optional notes |
| `listOrder` | Int | Order in the route stack |
| `timestamp` | Long | Created date |

### DAO Requirements

- `getAllInvoices(): Flow<List<InvoiceRecord>>`
- `getInvoiceById(id): Flow<InvoiceRecord>`
- `insertOrUpdate(invoice)`
- `delete(invoice)`

---

## 6. OCR Pipeline – From Camera to Parsed Fields

1. User captures invoice using **CameraX**.
2. Bitmap passed to **ML Kit TextRecognizer**.
3. Raw text processed through `InvoiceParser`:
   - `extractInvoiceNumber`
   - `extractPhoneNumber`
   - `extractAddress`
   - `extractCustomerName`
4. Parsed fields become a temporary `InvoiceRecord`.
5. User confirms/edit on **Detail Screen**.

---

## 7. UX Architecture

### 7A. Home Screen – "Route Stack"

A dense, minimal, scrollable list of delivery cards:

- **Invoice #** (Green accent)
- **Customer Name** (Primary large text)
- **Address** (Secondary text)
- **Status Chip**
- **Service Type Chips**

**Interactions:**

- **Long Press** → Drag to reorder route
- **Tap** → Open detail (Digital Clipboard)

> No horizontal scrolling.  
> No spreadsheet tables.  
> The list IS your "master view".

### 7B. Detail Screen – "Digital Clipboard"

This screen mimics a physical delivery packet:

1. **Top:** Invoice Image (zoomable)
2. **Middle:** Items list + service type
3. **Immediately Below:** Signature Pad ("Sign for receipt of items above")
4. **Bottom:** Notes

**Floating Actions (FABs):**

- **Maps** → Navigate to address
- **Phone** → Dial customer

---

## 8. Proof of Delivery (POD)

- Custom **CameraX** capture
- Saved to internal storage
- Thumbnail displayed in Detail Screen
- Tap to enlarge

---

## 9. Signature Capture

A **Compose Canvas** with `detectDragGestures`:

- Draw strokes
- Clear button
- Save to PNG
- Store URI in Room

Placed directly beneath item list so the customer sees **WHAT they are signing**.

---

## 10. Export Engine

Triggered from Home Screen top bar:

1. Query today's invoices
2. Generate CSV to `cacheDir`
3. Provide via FileProvider
4. Open Android Sharesheet (Gmail, Drive, Slack, etc.)

---

## 11. Development Workflow (High Level)

| Phase | Goal | Deliverables |
|-------|------|-------------|
| Phase 1 | Foundation | Room + Hilt + Entities + DAO |
| Phase 2 | Capture/OCR | CameraX, ML Kit, Parser utils |
| Phase 3 | Interface | HomeScreen, DetailScreen, Monokai theme, drag-drop |
| Phase 4 | Polish | Signature pad, export engine, FAB intents |

---

## 12. AI Coding Prompts (Summary)

Each stage of development has its own strict prompt.

> These are delivered in the **WORKFLOW.md** document.

---

## 13. Purpose of This Document

This README acts as the **single source of truth** for:

- AI coding agents
- Human developers
- Future maintainers

It defines architecture, data, business logic, UX, and tool requirements.
