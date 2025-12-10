# DAILY DOCKET – DEVELOPER ONBOARDING GUIDE

> A structured guide to get any developer productive in under 30 minutes.

---

## 1. What Daily Docket Is

Daily Docket is a **Native Android, offline-first delivery workflow system**. Drivers use the app to:

- OCR invoices using the device camera
- Confirm and adjust delivery details
- Capture Proof of Delivery (POD) photos
- Capture customer signatures
- Reorder their delivery route
- Navigate and call customers
- Export the daily delivery log

The app is built for **speed, resilience, and reliability** in environments with no network connectivity.

---

## 2. Technical Foundations

### You must understand these technologies

| Area | Tool | Purpose |
|------|------|---------|
| Architecture | MVVM + Clean Architecture | Organizes logic, data, UI |
| UI | Jetpack Compose | Declarative UI |
| Theme | Monokai Dimmed | High contrast for field work |
| Database | Room | Offline-first storage |
| OCR | ML Kit Text Recognition | Fast, on-device invoice scanning |
| Camera | CameraX | Embedded camera preview and capture |
| Images | Coil | Load/display POD + signature png |
| DI | Hilt | Dependency injection |
| Async | Coroutines + Flow | Background work, streams |
| Navigation | Navigation Compose | Screen transitions |

If you know these, you can develop confidently.

---

## 3. System Structure

```
/app
 ├── data
 │    ├── local
 │    │     ├── InvoiceRecord.kt
 │    │     ├── InvoiceDao.kt
 │    │     ├── DailyDocketDatabase.kt
 │    ├── repository
 │    │     ├── InvoiceRepository.kt
 │    │     ├── InvoiceRepositoryImpl.kt
 │
 ├── di
 │    ├── AppModule.kt
 │
 ├── ui
 │    ├── theme
 │    │     ├── Color.kt
 │    │     ├── Theme.kt
 │    ├── home
 │    │     ├── HomeScreen.kt
 │    ├── detail
 │    │     ├── DetailScreen.kt
 │    ├── camera
 │          ├── CameraScreen.kt
 │          ├── TextRecognitionAnalyzer.kt
 │          ├── InvoiceParser.kt
 │
 ├── viewmodel
 │    ├── HomeViewModel.kt
 │    ├── DetailViewModel.kt
 │
 ├── export
 │    ├── CsvExporter.kt
 │    ├── FileProviderConfig.xml
 │
 ├── MainActivity.kt
 │
 └── README.md
```

---

## 4. Development Workflow (How to Contribute)

### 1. Phase-based Development

Daily Docket is built in strict phases:

1. **Foundation** → Database + Hilt
2. **OCR + Camera** → Core capture pipeline
3. **UI** → Route Stack + Digital Clipboard
4. **Polish** → Signature saving + Export

> ⚠️ You may not jump ahead — each phase depends on the previous one.

### 2. Single Source of Truth

The `README.md` defines:

- Architecture
- Schema
- Tooling
- UX and UI details
- Theme and color standards

**All pull requests must reference it.**

---

## 5. How Data Flows Through Daily Docket

```
CameraX → Bitmap → ML Kit OCR → Parser → Temp Invoice
 → Detail Screen (edit/verify)
 → Save → Room DB → Home Screen Route Stack
 → Signature → Update status
 → CSV Export
```

This is the entire life cycle of a delivery.

---

## 6. Testing Checklist

When adding a feature:

- ✅ Does the app still work offline?
- ✅ Are DB updates visible on the Home screen?
- ✅ Does OCR still parse after code changes?
- ✅ Does signature saving produce correct PNGs?
- ✅ Does drag-and-drop reorder persist correctly?
- ✅ Does the Monokai theme remain consistent?
- ✅ Does your change break the CSV export?

---

## 7. Design Philosophy

- **Minimalism + clarity**
- Zero horizontal scrolling (portrait optimized)
- Cards = primary data containers
- Signature under invoice & items
- Drivers must complete deliveries with one hand
- High contrast, high legibility
- Minimize taps to complete a job

---

## 8. Getting Started in Android Studio

### Step 1 — Clone repo

### Step 2 — Open in Android Studio

### Step 3 — Build Gradle

Let dependencies download.

### Step 4 — Run the app on a physical device

> ⚠️ **CameraX and ML Kit do NOT work on emulators.**

---

## 9. How to Work With AI Coding Tools

When prompting:

- Always reference the **README**
- Use **phase-based prompts**
- Never ask AI to generate full app in a single prompt
- Build one file or component at a time
- Test after each phase

---

## 10. Common Pitfalls

| Issue | Fix |
|-------|-----|
| CameraX not showing preview | Use real device + correct lifecycle |
| ML Kit fails | Ensure bitmap rotation is correct |
| Reorder not persisting | Update `listOrder` in database |
| Signature blank | Ensure Canvas path is saved to PNG |
| Export not sharing | Configure FileProvider |

---

## ✔️ You are now ready for development

---

# 🚀 DAILY DOCKET – KICKSTART CODE FRAMEWORK (PHASE 1 SCAFFOLD)

> This provides the actual file structure + minimal empty code needed to start Phase 1.
>
> All files compile but contain only structure (no business logic yet).  
> This is the safe foundation for any AI tool to build on.

---

## 1. build.gradle (Module) – Dependency Shell

```gradle
dependencies {

    // Kotlin
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3"

    // Hilt
    implementation "com.google.dagger:hilt-android:2.48"
    ksp "com.google.dagger:hilt-compiler:2.48"

    // Room
    implementation "androidx.room:room-runtime:2.6.1"
    implementation "androidx.room:room-ktx:2.6.1"
    ksp "androidx.room:room-compiler:2.6.1"

    // CameraX
    implementation "androidx.camera:camera-core:1.3.1"
    implementation "androidx.camera:camera-camera2:1.3.1"
    implementation "androidx.camera:camera-lifecycle:1.3.1"
    implementation "androidx.camera:camera-view:1.3.1"

    // ML Kit
    implementation "com.google.android.gms:play-services-mlkit-text-recognition:18.1.0"

    // Coil
    implementation "io.coil-kt:coil-compose:2.4.0"

    // Gson
    implementation "com.google.code.gson:gson:2.10.1"

    // Compose
    implementation platform("androidx.compose:compose-bom:2024.01.00")
    implementation "androidx.compose.material3:material3"
    implementation "androidx.compose.ui:ui"
    implementation "androidx.compose.ui:ui-tooling-preview"

    // Navigation
    implementation "androidx.navigation:navigation-compose:2.7.0"
}
```

---

## 2. InvoiceRecord.kt (Entity Skeleton)

```kotlin
@Entity(tableName = "invoices")
data class InvoiceRecord(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val invoiceNumber: String = "",
    val customerName: String = "",
    val customerAddress: String = "",
    val customerPhone: String = "",
    val items: String = "",
    val status: String = "Open",
    val podImagePath: String? = null,
    val signaturePath: String? = null,
    val driverNotes: String = "",
    val listOrder: Int = 0,
    val timestamp: Long = System.currentTimeMillis()
)
```

---

## 3. InvoiceDao.kt (DAO Skeleton)

```kotlin
@Dao
interface InvoiceDao {

    @Query("SELECT * FROM invoices ORDER BY listOrder ASC")
    fun getAllInvoices(): Flow<List<InvoiceRecord>>

    @Query("SELECT * FROM invoices WHERE id = :id")
    fun getInvoiceById(id: Long): Flow<InvoiceRecord>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertOrUpdate(invoice: InvoiceRecord)

    @Delete
    suspend fun delete(invoice: InvoiceRecord)
}
```

---

## 4. DailyDocketDatabase.kt

```kotlin
@Database(
    entities = [InvoiceRecord::class],
    version = 1,
    exportSchema = true
)
abstract class DailyDocketDatabase : RoomDatabase() {
    abstract fun invoiceDao(): InvoiceDao
}
```

---

## 5. AppModule.kt (Hilt DI)

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): DailyDocketDatabase {
        return Room.databaseBuilder(
            context,
            DailyDocketDatabase::class.java,
            "daily_docket_db"
        ).build()
    }

    @Provides
    @Singleton
    fun provideInvoiceDao(db: DailyDocketDatabase): InvoiceDao = db.invoiceDao()

    @Provides
    @Singleton
    fun provideGson(): Gson = Gson()
}
```

---

## 6. Repository Framework

### InvoiceRepository.kt

```kotlin
interface InvoiceRepository {
    fun getAllInvoices(): Flow<List<InvoiceRecord>>
    fun getInvoiceById(id: Long): Flow<InvoiceRecord>
    suspend fun save(invoice: InvoiceRecord)
    suspend fun delete(invoice: InvoiceRecord)
}
```

### InvoiceRepositoryImpl.kt

```kotlin
@Singleton
class InvoiceRepositoryImpl @Inject constructor(
    private val dao: InvoiceDao
) : InvoiceRepository {

    override fun getAllInvoices() = dao.getAllInvoices()

    override fun getInvoiceById(id: Long) = dao.getInvoiceById(id)

    override suspend fun save(invoice: InvoiceRecord) {
        dao.insertOrUpdate(invoice)
    }

    override suspend fun delete(invoice: InvoiceRecord) {
        dao.delete(invoice)
    }
}
```

---

## ✔️ You now have

- ✅ A full onboarding guide
- ✅ A Phase 1 code framework that is safe to expand
- ✅ The architecture and workflow ready for AI-assisted building
