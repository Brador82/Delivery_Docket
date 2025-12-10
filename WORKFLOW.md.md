# Daily Docket – Development Workflow & AI Build Prompts

> **Use this after the README.**  
> This is the actionable, step-by-step build order for the entire application.

## Overview

Each Phase includes:

- **What must be built**
- **Why it exists**
- **Constraints**
- **AI-ready directive prompt**
- **Expected deliverables**

---

## 🌑 Phase 1 — FOUNDATION

**Goal:** Build the project skeleton, database layer, dependency injection, and groundwork for all later features.

**Why:** Daily Docket is offline-first. The database must exist before UI or OCR.

### Phase 1 — Requirements

You must implement:

#### 1. Gradle Dependencies

Add all tools defined in the README section "Technology Stack & Requirements":

- Kotlin / Coroutines / Flow
- Hilt
- Room (runtime + ktx + compiler)
- CameraX
- ML Kit Text Recognition v2
- Coil
- Gson
- Navigation Compose
- Material 3

#### 2. Database

Create:

- `InvoiceRecord.kt` (Room Entity)
- `InvoiceDao.kt`
- `DailyDocketDatabase.kt` (abstract Room DB)

#### 3. DI Layer (Hilt)

Create:

- `AppModule.kt` providing:
  - Room database
  - DAO
  - Gson instance

#### 4. Core Repository Interface

Create:

- `InvoiceRepository.kt` interface
- `InvoiceRepositoryImpl.kt` implementation

(This will be used later by ViewModels.)

### Phase 1 — AI Coding Prompt

Copy/paste below into your coding agent:

```
PHASE 1 DIRECTIVE — FOUNDATION (DO NOT SKIP ANY REQUIREMENT)

Reference: Use the Daily Docket README as the architectural source of truth.

Implement Phase 1 of the Daily Docket app:

1. Update build.gradle (Module) to include ALL dependencies listed in the README 
   under "3. Technology Stack & Requirements."

2. Create the Room entity InvoiceRecord using the exact schema defined in 
   Section 4 of the README.

3. Create InvoiceDao with:
   - getAllInvoices(): Flow<List<InvoiceRecord>>
   - getInvoiceById(id: Long): Flow<InvoiceRecord>
   - insertOrUpdate(invoice: InvoiceRecord)
   - delete(invoice: InvoiceRecord)

4. Create the Room database DailyDocketDatabase.

5. Implement Hilt DI:
   - Create AppModule
   - Provide: Room DB, DAO, Gson.

6. Create InvoiceRepository and InvoiceRepositoryImpl wired through Hilt.

7. Provide complete, working Kotlin code files for all components.

Do NOT write UI, ViewModels, or Camera code yet.
```

### Phase 1 — Expected Deliverables

- ✅ Fully compiling Room DB
- ✅ Hilt wired correctly
- ✅ Repository established
- ✅ Project builds successfully
- ❌ No UI yet

---

## 📸 Phase 2 — CAPTURE & OCR ("The Eyes")

**Goal:** Implement CameraX + ML Kit flow to extract text from invoices.

**Why:** OCR is the core automation feature.

### Phase 2 — Requirements

#### 1. CameraX Integration

Create:

- `CameraScreen.kt` (Compose)
  - Live preview
  - Capture button
  - Function returning Bitmap (or ImageProxy → Bitmap)

#### 2. OCR Engine

Create:

- `TextRecognitionAnalyzer.kt` → handles ML Kit processing
  - Runs OCR entirely offline
  - Returns raw text

#### 3. Regex Parsing

Create:

- `InvoiceParser.kt` with strict, production-ready regex:
  - `extractInvoiceNumber(text)`
  - `extractPhoneNumber(text)`
  - `extractAddress(text)`
  - `extractCustomerName(text)`

#### 4. Temporary Invoice Builder

Repository method:

```kotlin
suspend fun processImage(bitmap): InvoiceRecord
```

This produces a temporary record filled with extracted fields (unconfirmed).

### Phase 2 — AI Coding Prompt

```
PHASE 2 DIRECTIVE — CAPTURE & OCR

Reference: Use the schema & behavior defined in the README.

Implement the OCR Pipeline:

1. Build a CameraScreen.kt that uses CameraX with:
   - PreviewView via Compose
   - Capture button
   - Returns a Bitmap

2. Create TextRecognitionAnalyzer class that:
   - Accepts Bitmap or ImageProxy
   - Uses ML Kit TextRecognizer v2
   - Returns raw text

3. Create InvoiceParser.kt with production regex functions:
   - extractInvoiceNumber
   - extractCustomerName
   - extractAddress
   - extractPhoneNumber

4. In InvoiceRepositoryImpl, add:
   suspend fun processImage(bitmap: Bitmap): InvoiceRecord
   - Run OCR
   - Parse using InvoiceParser
   - Return a temporary InvoiceRecord (not saved yet)

5. Provide complete Kotlin code for all components.

Do NOT write UI editing screens yet.
```

### Phase 2 — Expected Deliverables

- ✅ Functional CameraX capture
- ✅ OCR returning near-instant text
- ✅ Regex parsing pipeline
- ✅ Preview functions included
- ❌ No UI beyond camera preview

---

## 🖥️ Phase 3 — INTERFACE (Smart List + Digital Clipboard)

**Goal:** Build the full UI using Compose.

**Why:** This is how the user interacts with all features.

### Phase 3 — Requirements

#### 1. Navigation Setup

Create:

```kotlin
NavHost(
    startDestination = "home",
    routes: home, detail/{id}, camera
)
```

#### 2. Home Screen — "Route Stack"

A dense, reorderable list with:

- Invoice # (green)
- Customer Name
- Address (first line)
- Status chip
- Service type chips (Install, Drop-Off, etc.)
- Drag-and-Drop reordering (using standard Compose reorder libraries or manual implementation)
- FAB → Add New Delivery (opens Camera)

#### 3. Detail Screen — "Digital Clipboard"

Top-to-bottom flow mimicking paper workflow:

- Invoice Image (zoomable)
- Items list + service type
- Signature Pad directly beneath item list
- Notes
- Save button (updates Room)
- Floating FABs:
  - Maps → navigate to customer
  - Phone → dial customer

#### 4. Theme Implementation

Monokai Dimmed with:

- `Color.kt`
- `Theme.kt`
- Typography
- Surface elevations
- Shadows/elevation applied exactly as described

### Phase 3 — AI Coding Prompt

```
PHASE 3 DIRECTIVE — INTERFACE (SMART LIST + DIGITAL CLIPBOARD)

Reference: Follow the README section "UX Architecture".

Implement the full UI:

1. Create Navigation using NavHost (home, detail/{id}, camera).

2. Build HomeScreen:
   - LazyColumn of cards
   - Card layout uses Monokai palette
   - Includes service chips
   - Long-press to reorder cards
   - Tap to open DetailScreen

3. Build DetailScreen:
   - Show invoice image at top
   - Below: items + service type
   - Below that: signature pad (Canvas + gestures)
   - Notes field
   - Save button that updates Room

4. Add floating FAB tools:
   - Maps Intent
   - Phone Intent

5. Implement Monokai Dimmed theme using exact colors from the README.

6. Provide ALL code:
   - HomeScreen.kt
   - DetailScreen.kt
   - Theme.kt
   - Color.kt
   - Previews for each component

Do NOT implement export or signature saving logic yet (Phase 4).
```

### Phase 3 — Expected Deliverables

- ✅ Fully themed UI
- ✅ Smart List route screen
- ✅ Detail screen with signature + item info
- ✅ Navigation wired
- ❌ No export yet

---

## 📝 Phase 4 — POLISH (Signatures, Export, Intents)

**Goal:** Finish all high-value business features.

**Why:** These finalize the complete delivery workflow.

### Phase 4 — Requirements

#### 1. Signature Pad Saving

- Convert Canvas strokes to Bitmap
- Save PNG locally
- Write path to Room
- Auto-update status ("Delivered") after signature saved

#### 2. Export Engine

Create a CSV generator:

- Query today's invoices
- Convert to CSV string
- Save file to `cacheDir`
- Expose via FileProvider
- Launch Android Sharesheet

#### 3. Final Intents

- `geo:0,0?q=` maps navigation
- `tel:` phone dialing

### Phase 4 — AI Coding Prompt

```
PHASE 4 DIRECTIVE — POLISH & EXPORT

Reference: Use README sections "Signature Capture" and "Export Engine."

Implement:

1. Signature Pad saving pipeline:
   - Capture path
   - Save PNG to internal storage
   - Store path in Room
   - Mark invoice as Delivered

2. Export Engine:
   - Query today's invoices
   - Convert to CSV
   - Save to cacheDir
   - Share via FileProvider

3. Finalize FAB Intents:
   - Maps FAB → Launch geo Intent
   - Phone FAB → Launch Dialer

4. Provide complete Kotlin code for all exports, file providers, 
   signature saving, and intents.
```

### Phase 4 — Expected Deliverables

- ✅ Signature saved, stored, and displayed
- ✅ Export working
- ✅ Entire delivery cycle functional end-to-end

---

## ✔️ Final Output of This Workflow

Once Phases 1–4 are complete, you will have:

- ✅ A fully functional offline-first Android application
- ✅ OCR → Items → Route → Delivery → Signature → Export
- ✅ Production-ready UI
- ✅ Clean architecture with Hilt
- ✅ Reliable data flow through Room
- ✅ Professional theme (Monokai Dimmed)
- ✅ A system any delivery team can use in real time
