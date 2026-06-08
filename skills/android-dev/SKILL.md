---
name: android-dev
description: >
  Comprehensive Android app development skill covering all major development paths — native Android (Java/Kotlin), cross-platform (Flutter, React Native, KMM, Compose Multiplatform), and hybrid (Capacitor/Ionic) — but explicitly excluding iOS-only workflows. Use this skill whenever the user asks about building, designing, architecting, debugging, testing, releasing, or maintaining any Android or cross-platform mobile app. Trigger on any mention of: Android, Kotlin, Jetpack Compose, Flutter, React Native, KMM, APK, Play Store, Gradle, mobile app development, Android Studio, cross-platform mobile, or any Android-related library/framework. Always use this skill even if the request seems simple — it encodes battle-tested big-tech engineering practices for the full dev lifecycle.
---

# Android App Development Skill

This skill guides production-grade Android and cross-platform (non-iOS) app development following practices used at big tech companies. It covers the entire development lifecycle — architecture, UI, code quality, testing, error handling, release, and maintenance.

## Quick Reference — What to Read

| User asks about... | Read |
|--------------------|------|
| Which stack to pick | §1 Stack Selection |
| Architecture, project structure | §2 Architecture |
| UI, screens, design system | §3 UI & Design |
| Code quality, patterns, APIs | §4 Code Quality |
| Error handling, crashes | §5 Error Handling |
| Testing strategy | §6 Testing |
| Build, CI/CD, release | §7 Build & Release |
| Performance, memory | §8 Performance |
| Debugging, bug fixing | §9 Debugging |
| Full roadmap | §10 Development Roadmap |
| Deep reference for a stack | See `references/` directory |

---

## §1 Stack Selection

Choose based on team, requirements, and platform targets. **Do not recommend iOS-specific paths.**

### Native Android — Kotlin + Jetpack Compose
**Best for:** Android-only apps, hardware-intensive features, best-in-class UX, new projects.
- Language: **Kotlin**
- UI: **Jetpack Compose** (modern declarative UI)
- Key libs: Room, Retrofit/Ktor, Hilt, WorkManager, DataStore, Navigation Compose
- Reference: `references/native-android.md`

### Native Android — Java + XML Views
**Best for:** Existing Java codebases, teams without Kotlin experience, legacy app maintenance, incremental Kotlin migration.
- Language: **Java** (fully supported by Google, not deprecated)
- UI: **XML Layouts** (ConstraintLayout, RecyclerView, ViewBinding)
- Key libs: Room, Retrofit, Hilt, WorkManager, LiveData, ViewModel
- Java and Kotlin **coexist seamlessly** in the same project — migrate incrementally
- Reference: `references/java-android.md`

### Flutter (Dart)
**Best for:** Android + Web (+ desktop) from one codebase, fast iteration, pixel-perfect custom UI.
- Language: **Dart**
- UI: Flutter Widget tree (Material 3 / Cupertino widgets available but target Material for Android)
- Key libs: Provider/Riverpod/Bloc, Dio, Drift/Isar, go_router, flutter_local_notifications
- Reference: `references/flutter.md`

### React Native (JavaScript/TypeScript)
**Best for:** Web + Android code sharing, JS/TS teams, rich ecosystem.
- Language: **TypeScript** (preferred)
- UI: React Native core components + NativeWind / React Native Paper
- Key libs: React Navigation, Zustand/Redux Toolkit, React Query, MMKV
- Reference: `references/react-native.md`

### Kotlin Multiplatform (KMM / Compose Multiplatform)
**Best for:** Sharing business logic across Android + Desktop + Web while keeping native Android UI.
- Language: **Kotlin** everywhere
- UI: Native Compose on Android; Compose Multiplatform for shared UI
- Key libs: Ktor, SQLDelight, Koin, kotlinx.serialization, Napier
- Reference: `references/kmm.md`

### Hybrid (Capacitor / Ionic)
**Best for:** Web-first teams, simple apps, PWA-like content apps.
- Language: TypeScript + HTML/CSS
- UI: Ionic components or custom web UI
- Avoid for: Heavy animations, native sensor access, high-performance games
- Reference: `references/hybrid.md`

### Decision Matrix

| Requirement | Native Kotlin | Native Java | Flutter | RN | KMM | Hybrid |
|---|---|---|---|---|---|---|
| Android-only (new) | ✅ Best | ✅ | ✅ | ✅ | ✅ | ✅ |
| Android-only (existing Java) | ⚠️ migrate | ✅ Best | ❌ | ❌ | ⚠️ | ❌ |
| Android + Web | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ Best |
| Android + Desktop | ❌ | ❌ | ✅ | ⚠️ | ✅ | ⚠️ |
| Shared business logic only | N/A | N/A | N/A | N/A | ✅ Best | N/A |
| Native performance | ✅ | ✅ | ✅ | ⚠️ | ✅ | ❌ |
| JS/TS team | ❌ | ❌ | ❌ | ✅ Best | ❌ | ✅ |
| Custom pixel-perfect UI | ✅ | ⚠️ | ✅ Best | ⚠️ | ✅ | ❌ |

---

## §2 Architecture

### Core Principle: Separation of Concerns
Every production Android project must separate **UI**, **business logic**, and **data** into distinct, independently testable layers.

### Recommended Architecture: Clean Architecture + MVI/MVVM

```
app/
├── ui/              # Composables / Activities / Fragments / Screen states
├── presentation/    # ViewModels, UI State, UI Events
├── domain/          # Use cases, domain models, repository interfaces
├── data/            # Repository impl, remote (API), local (DB), mappers
└── di/              # Dependency injection modules
```

**Data flow (unidirectional):**
```
User Action → ViewModel/Store → Use Case → Repository → Data Source
                    ↓
             UI State (sealed class / StateFlow)
                    ↓
             Composable / View renders state
```

### Key Architecture Patterns by Stack

**Native (MVVM + MVI):**
- `StateFlow` / `SharedFlow` for reactive state
- `sealed class UiState` + `sealed class UiEvent`
- Hilt for DI, coroutines + Flow for async
- Repository pattern wrapping Room + Retrofit

**Flutter (BLoC or Riverpod):**
- `Bloc` or `Cubit` for business logic isolation
- `AsyncNotifierProvider` (Riverpod) for data + state
- Repositories as abstract classes with impl injected

**React Native (Redux Toolkit or Zustand):**
- RTK Query or React Query for server state
- Zustand slices for client state
- Custom hooks to encapsulate business logic per feature

**KMM:**
- Shared `commonMain` holds domain + data layers
- `expect/actual` for platform-specific implementations
- Kotlin coroutines + Flow bridged to platform (StateFlow on Android)

### Module Structure (Multi-module for large apps)

```
:app            # Entry point, DI wiring
:core:ui        # Design system, shared composables
:core:network   # API client, interceptors
:core:database  # Room / SQLDelight setup
:feature:home
:feature:profile
:feature:settings
```

---

## §3 UI & Design

### Design System First
Before writing screens, define:
1. **Color tokens** — Primary, secondary, surface, on-surface, error; light + dark variants
2. **Typography scale** — Display, headline, title, body, label (Material 3 type system)
3. **Spacing scale** — 4dp grid system (4, 8, 12, 16, 24, 32, 48dp)
4. **Shape tokens** — Corner radii per component family
5. **Component library** — Button, TextField, Card, BottomSheet, TopAppBar, etc.

### Jetpack Compose UI Rules
- Use `MaterialTheme` tokens; never hardcode colors/dimensions
- `CompositionLocal` for theme, locale, haptics
- `remember` / `rememberSaveable` correctly (saveable for UI state surviving rotation)
- Extract large composables into sub-composables; each function ≤ 80 lines
- Use `LazyColumn`/`LazyVerticalGrid` for lists; never `Column` with forEach for large data
- Side effects only in `LaunchedEffect`, `DisposableEffect`, `SideEffect`
- Avoid state hoisting anti-patterns: hoist state to the lowest common ancestor

### Accessibility (Non-Negotiable)
- All interactive elements: `contentDescription` or `semantics { }`
- Min touch target: **48×48dp**
- `TalkBack` compatibility tested before every release
- Dynamic text size support (`sp` not `dp` for text)
- Color contrast ratio ≥ 4.5:1 (WCAG AA)

### Navigation
- **Native:** Navigation Compose with typed `NavHost` and `SafeArgs` equivalent
- **Flutter:** `go_router` with named routes and guards
- **RN:** React Navigation v7 with typed `NavigationProp`
- Deep link handling registered for every screen that can be externally opened
- Back stack managed deliberately — don't push duplicates, use `popUpTo` / `launchSingleTop`

### Responsive & Adaptive UI
- Support all screen sizes: phones, foldables, tablets (`WindowSizeClass`)
- Test at 320dp, 360dp, 411dp, 600dp+, 840dp+ widths
- Foldable hinge awareness via `WindowInfoTracker`
- Edge-to-edge display + `WindowInsets` handling required for Android 15+

---

## §4 Code Quality

### Language Standards

**Kotlin:**
- Prefer `data class`, `sealed class`, `object`, `enum class` appropriately
- No `!!` null assertions — use `?.let`, `?: return`, `requireNotNull` with message
- Coroutines: always specify `CoroutineScope` + `Dispatcher` explicitly; never `GlobalScope`
- Use `@Stable` / `@Immutable` on Compose state classes for smart recomposition

**Java:**
- `@NonNull` / `@Nullable` annotations on every method param and return type
- Never call methods on unchecked objects — null-check explicitly or use `Objects.requireNonNull`
- Always null `binding` reference in Fragment's `onDestroyView()` to prevent memory leaks
- Use `ExecutorService` (not `AsyncTask` — deprecated) for background work; or `LiveData` + Room's built-in threading
- Prefer `ListAdapter` + `DiffUtil` over manual `notifyDataSetChanged()` in RecyclerView
- Use `ViewBinding` — never `findViewById`

**Dart (Flutter):**
- Null safety required — no `!` without explicit null check above
- Immutable state objects with `copyWith`
- `const` constructors on all stateless widgets

**TypeScript (RN):**
- `strict: true` in tsconfig always
- Zod or io-ts for runtime type validation of API responses
- No `any` — use `unknown` and narrow

### Dependency Management
- Pin all dependency versions in `build.gradle.kts` / `pubspec.yaml` / `package.json`
- Audit dependencies monthly for security vulnerabilities
- Avoid transitive dependency conflicts — use dependency resolution strategies
- Keep dependency count minimal — every added lib is a maintenance burden

### Code Review Checklist (PR gate)
- [ ] New public APIs have KDoc / DartDoc / JSDoc
- [ ] No hardcoded strings — use string resources / l10n
- [ ] No hardcoded dimensions or colors outside design tokens
- [ ] No blocking I/O on main thread
- [ ] No memory leaks (no `Activity` context stored in singletons)
- [ ] Coroutine scopes / streams properly cancelled / disposed
- [ ] Feature flag guarding any non-trivial feature

---

## §5 Error Handling

### The Golden Rule
**Never let exceptions propagate to the user silently or crash the app.**

### Error Classification

| Type | Strategy |
|------|----------|
| Network errors | Retry with exponential backoff; show retry UI |
| Auth errors (401/403) | Refresh token → re-request → logout if fails |
| Validation errors | Show inline field errors immediately |
| Data parsing errors | Log + fallback to cached/default state |
| Unexpected crashes | Catch at top-level; show error screen + report |
| Background task failures | Retry via WorkManager; notify user if critical |

### Result / Either Pattern (Kotlin)
```kotlin
sealed class AppResult<out T> {
    data class Success<T>(val data: T) : AppResult<T>()
    data class Error(val exception: AppException) : AppResult<Nothing>()
}

sealed class AppException(msg: String) : Exception(msg) {
    class NetworkException(msg: String) : AppException(msg)
    class AuthException(msg: String) : AppException(msg)
    class ParseException(msg: String) : AppException(msg)
    class UnknownException(msg: String) : AppException(msg)
}
```

Use `AppResult<T>` as return type for all repository + use case functions. ViewModels map to `UiState.Error`.

### Crash Reporting
- Integrate **Firebase Crashlytics** or **Sentry** from day one
- Set user identifiers and custom keys before crash occurs
- Non-fatal exceptions logged for all caught errors
- ANR monitoring enabled
- Crash-free sessions target: **≥ 99.5%**

### Offline / Network Resilience
- Cache-first strategy: show stale data, fetch fresh in background
- `Room` / `Drift` / `MMKV` as single source of truth
- Expose network state via `ConnectivityManager` and reflect in UI
- All network calls wrapped with timeout + retry policy

---

## §6 Testing

### Testing Pyramid

```
         /\
        /E2E\        ← 10%  (UI tests: Espresso, Maestro, Appium)
       /------\
      / Integr \     ← 20%  (Repository, DB, API contract tests)
     /----------\
    /    Unit    \   ← 70%  (ViewModels, Use Cases, Utilities)
   /--------------\
```

### Unit Tests (70%)
- Every ViewModel, UseCase, Repository, Mapper tested
- **Native:** JUnit5 + MockK + Turbine (Flow testing) + Kotest assertions
- **Flutter:** `flutter_test` + `mocktail`
- **RN:** Jest + `@testing-library/react-native` + `msw` for API mocking
- Coverage target: **≥ 80%** on domain + presentation layers

### Integration Tests (20%)
- Room DB tests with in-memory database
- Retrofit/Ktor tests with `MockWebServer` (OkHttp)
- Repository tests verifying cache + remote coordination
- API contract tests against real staging endpoint

### UI / E2E Tests (10%)
- **Espresso** for critical user journeys (login, checkout, core action)
- **Maestro** for cross-platform E2E flows (recommended for Flutter + RN too)
- Run on real device farm (Firebase Test Lab / BrowserStack) before release
- Smoke test suite runs on every PR; full E2E suite nightly

### Test Data Management
- Use factories / builders for test data, never copy-paste objects
- Hermetic tests: never share mutable state between test cases
- Fakes over mocks for complex dependencies (repositories, data sources)

---

## §7 Build & Release

### Build Variants
```
debug       → dev API, logging on, no minification, debuggable
staging     → staging API, logging on, minified, not debuggable  
release     → prod API, logging off, minified, signed
```

### Gradle Best Practices (Native)
- `build.gradle.kts` only — no Groovy DSL in new projects
- Version catalog (`libs.versions.toml`) for all dependency versions
- `buildConfig` for environment-specific constants
- Baseline profiles for startup performance
- R8 full mode enabled in release; maintain proguard rules in version control

### CI/CD Pipeline

```
PR Opened
  └─ lint + unit tests + build debug APK          [< 5 min]

Merge to main
  └─ unit + integration tests + staging build     [< 15 min]
  └─ deploy to Firebase App Distribution (QA)

Release tag
  └─ full test suite + E2E on device farm         [< 45 min]
  └─ build release AAB
  └─ upload to Play Console (internal track)
  └─ promote: internal → closed testing → open → production
```

**Recommended CI:** GitHub Actions, Bitrise, or CircleCI.

### Play Store Release Strategy
- Always release to **internal → closed → open testing** before production
- Use **staged rollouts**: 5% → 20% → 50% → 100% with 24-48h monitoring
- Monitor Crashlytics + ANR rate + rating before expanding rollout
- **Never skip staged rollout** for significant changes

### App Signing
- Upload key (Play App Signing): stored in CI secrets, never committed
- Use Google Play App Signing for distribution key management
- Document key recovery procedure in team runbook

---

## §8 Performance

### Startup Performance
- App startup time target: **cold start < 1s**, warm start < 500ms
- Use **App Startup library** for initializing libraries lazily
- Baseline profiles generated + committed to repo
- Heavy initialization moved off main thread

### UI Performance
- Target: **60fps** (90/120fps on supported devices); **zero jank**
- Measure with **Android Studio Profiler** + `FrameMetrics` API
- Avoid allocation in `draw()` / `onMeasure()` / composition
- Use `derivedStateOf` in Compose to avoid unnecessary recompositions
- Image loading: Coil (Compose) / Glide / Picasso — never load full-res in thumbnails

### Memory
- No `Activity` / `Context` references in ViewModels or singletons
- WeakReferences for listeners stored beyond their owner's lifecycle
- Bitmap recycling and memory cache sizing
- Heap dump + leak detection via **LeakCanary** in debug builds (always)

### Network
- HTTP caching headers respected
- Image CDN + WebP format
- Gzip/Brotli compression verified
- Request batching where applicable
- Connection pooling configured

### Battery
- Background work only via **WorkManager** with appropriate constraints
- Location updates: request only needed accuracy level; stop when backgrounded
- Wakelocks used sparingly with explicit release

---

## §9 Debugging & Bug Fixing

### Debugging Process

1. **Reproduce reliably** — document exact steps, device, OS version, account state
2. **Isolate** — is it UI, business logic, network, or persistence?
3. **Instrument** — add targeted logs / breakpoints, NOT shotgun logging
4. **Hypothesize** — form 1-3 specific hypotheses before touching code
5. **Fix the root cause** — never patch symptoms; trace back to the source
6. **Regression test** — write a test that fails before fix, passes after
7. **Document** — comment explaining why the fix works, not just what it does

### Common Android Bug Patterns

| Bug | Likely Cause | Fix |
|-----|-------------|-----|
| ANR | Main thread I/O / long computation | Move to coroutine/Dispatcher.IO |
| Memory leak | Context stored in singleton | Use `applicationContext`; WeakRef |
| Crash on rotation | ViewModel not used; state not saved | `rememberSaveable` / ViewModel |
| UI lag | Recomposition loops | `derivedStateOf`, stable params |
| Blank screen after API call | Error swallowed silently | Check error state propagation |
| Deep link not working | Manifest intent-filter missing | Verify `adb shell am start` test |
| Push notification silent | Background restrictions | Test on real devices across OEMs |

### Logging Standards
- **Production:** Firebase Crashlytics only (no `Log.d` in release builds)
- **Debug/Staging:** Timber with debug tree
- Log levels: ERROR (crashes), WARN (recoverable), INFO (key events), DEBUG (dev only)
- Never log PII — mask emails, phone numbers, tokens in logs

### OEM-Specific Issues
- Test on **Samsung**, **Xiaomi/MIUI**, **OnePlus/OxygenOS**, **Huawei (no GMS)** for critical flows
- Background restrictions vary widely by OEM — test push, alarms, background sync
- Maintain a physical or cloud device farm with top market-share devices

---

## §10 Development Roadmap

Follow this phase structure for any new Android project:

### Phase 0 — Foundation (Week 1-2)
- [ ] Stack decision documented with rationale
- [ ] Module structure defined
- [ ] Design system tokens defined (colors, type, spacing, shapes)
- [ ] CI pipeline running (lint + unit tests + build)
- [ ] Crash reporting integrated (Crashlytics/Sentry)
- [ ] Analytics baseline integrated (Firebase/Amplitude)
- [ ] API contract / mock server set up
- [ ] DI framework configured
- [ ] Navigation skeleton implemented
- [ ] Flavor/build variant config complete

### Phase 1 — Core Features (Weeks 3-8)
- [ ] Auth flow (login, register, token refresh, logout)
- [ ] Core screen shells with real navigation
- [ ] Network layer (client, interceptors, error handling)
- [ ] Local persistence layer (DB schema + DAOs)
- [ ] Repository layer wiring remote + local
- [ ] ViewModels + UI states for each feature
- [ ] Unit tests for all ViewModels + use cases
- [ ] Feature flags infrastructure

### Phase 2 — Polish (Weeks 9-12)
- [ ] Design QA pass against Figma/spec
- [ ] Accessibility audit (TalkBack, contrast, touch targets)
- [ ] Dark mode implementation + verification
- [ ] Localization (strings externalized, RTL support if needed)
- [ ] Loading, empty, error states on every screen
- [ ] Deep link handling
- [ ] Widget / notification implementation
- [ ] Offline mode verification

### Phase 3 — Hardening (Weeks 12-14)
- [ ] Performance profiling (startup, scroll, memory)
- [ ] E2E test suite on device farm (Firebase Test Lab)
- [ ] Security review (certificate pinning, biometrics, secure storage)
- [ ] Proguard / R8 rules verified
- [ ] Crash-free rate ≥ 99.5% on staging
- [ ] Play Store listing, screenshots, privacy policy

### Phase 4 — Release
- [ ] AAB signed and uploaded to internal track
- [ ] Staged rollout plan defined
- [ ] Monitoring dashboard set up (Crashlytics, Play Console vitals)
- [ ] Rollback plan documented
- [ ] On-call rotation assigned

### Phase 5 — Post-Launch (Ongoing)
- Crash-free rate monitored daily
- ANR rate < 0.47% (Play Store threshold)
- App rating monitored; negative reviews triaged weekly
- Dependency updates reviewed monthly
- OS beta testing with each new Android release

---

## Reference Files

For stack-specific deep dives, read:
- `references/native-android.md` — Kotlin, Compose, Room, Hilt, Coroutines
- `references/java-android.md` — Java, XML Views, ViewBinding, LiveData, Retrofit, Room, Hilt, migration path
- `references/flutter.md` — Dart, BLoC/Riverpod, Drift, go_router
- `references/react-native.md` — TypeScript, RN architecture, Hermes, New Architecture
- `references/kmm.md` — KMM shared modules, SQLDelight, Ktor, Compose Multiplatform
- `references/hybrid.md` — Capacitor, Ionic, PWA considerations