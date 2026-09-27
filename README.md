<div align="center">

# 🧮 ColorOS Calculator

**OPPO's polished ColorOS calculator, repackaged to run on more Android phones.**

A modified build of the stock ColorOS Calculator (v17.0.0) that ships the full feature set:
basic and scientific calculation, eight unit converters, live currency exchange rates,
a mortgage planner, a floating mini window, and lock screen cards.
This build fixes compatibility with Samsung devices and adds a custom dark-background animation.

[![Platform](https://img.shields.io/badge/platform-Android%2011%2B-3DDC84?logo=android&logoColor=white)](#getting-started)
[![Min SDK](https://img.shields.io/badge/minSdk-30%20(Android%2011)-3DDC84)](#getting-started)
[![Target SDK](https://img.shields.io/badge/targetSdk-36%20(Android%2016)-3DDC84)](#getting-started)
[![Version](https://img.shields.io/badge/version-17.0.0%20(17000000)-blue)](#whats-changed-in-this-build)
[![APK Size](https://img.shields.io/badge/APK-16.5%20MB-orange)](#inside-the-apk)
[![Language](https://img.shields.io/badge/language-Kotlin%20%2F%20Java-7F52FF?logo=kotlin&logoColor=white)](#tech-stack)

</div>

---

## 📌 What's changed in this build

This is a repackaged build of the ColorOS Calculator with two local modifications:

| # | Change | Where to see it |
|---|--------|-----------------|
| 1 | 📱 **Samsung compatibility fix.** The original build did not work on Samsung phones. This build adds Samsung device detection so the app initializes correctly on Galaxy devices. | `isSamsungDevice` check reading the `manufacturer` and `ro.product.brand.sub` system properties, inside `classes.dex` |
| 2 | 🌙 **Dark-background animation.** An added frame-based animation that plays over a dark background, with a dedicated night variant. | Frame sequences in `assets/images/` and `assets/images_night/`, plus the Lottie night files `assets/coui_lottie_*_night.json` |

Everything else is the authentic ColorOS Calculator experience, unpacked and re-signed
(v1 JAR signature plus a v2/v3 APK Signing Block) so it installs on any modern Android device.

## ✨ Features

### 🔢 Calculator core

| Feature | Details |
|---------|---------|
| Basic calculator | Tactile keypad with haptics (`VIBRATE`) and a key-press sound (`res/raw/calc_key.ogg`) |
| Scientific mode | Trig, logs, powers, constants (`π`, `e`), degree/radian modes, parentheses |
| Programmer view | Bit-level base conversion between binary, octal, decimal and hex (`BaseConvertActivity`) |
| Spoken results | Chinese voice announcements of digits and operators (`res/raw/cal_cn_*.wav`) |
| History and expressions | Expression evaluation engine with `oplus.intent.action.CALCULATOR` deep link |

### 🔄 Converters

| Converter | Entry activity | Intent action |
|-----------|----------------|---------------|
| Length | `LengthConvertActivity` | `coloros.intent.action.LENGTH_UNIT` |
| Area | `AreaConvertActivity` | `coloros.intent.action.AREA_UNIT` |
| Volume | `VolumeConvertActivity` | `coloros.intent.action.VOLUME_UNIT` |
| Weight | `WeightConvertActivity` | `coloros.intent.action.WEIGHT_UNIT` |
| Temperature | `TemperatureConvertActivity` | `coloros.intent.action.TEMPERATURE_UNIT` |
| Power | `PowerConvertActivity` | `coloros.intent.action.POWER_UNIT` |
| Pressure | `PressureConvertActivity` | `coloros.intent.action.PRESSURE_UNIT` |
| Speed | `SpeedConvertActivity` | `coloros.intent.action.SPEED_UNIT` |
| Currency (live rates) | `ExchangeRateActivity` | `coloros.intent.action.EXCHANGE_RATE` |

Conversion tables ship as data, not code: `assets/category_translation.json` (categories),
`assets/area_translation.json` and `assets/area_rate.json` (unit factors).

### 💰 Finance

| Feature | Details |
|---------|---------|
| Mortgage planner | `MortgageConvertActivity` with three loan types: commercial, provident fund, and portfolio loans |
| Loan checklist | `MortgageChecklistActivity` walks through the mortgage flow step by step |
| Data reset | `ClearDataReceiver` wipes mortgage data when the app's data is cleared |

### 🧩 System integration (ColorOS and beyond)

| Feature | Details |
|---------|---------|
| Floating mini window | `MiniActivity` + `FloatWindowService` render the calculator in a compact overlay (`SYSTEM_ALERT_WINDOW`) |
| Lock screen cards | Two card providers (`CalculatorDomesticLockScreenCardProvider`, `CalculatorExportLockScreenCardProvider`) with the card UI packaged in `assets/components.rpk` |
| Smart sidebar | Bracket-space integration; other apps can request expression evaluation via `com.oplus.calculator.EVALUATE_EXPRESSION` |
| System settings search | `SettingsSearchIndexablesProvider` surfaces calculator settings in system search |
| Global search indexing | Calculator results and data are exposed to device-wide search |
| Fast startup | Ships a baseline profile (`assets/dexopt/baseline.prof`) for AOT-compiled hot paths |

### 🎨 Experience

| Feature | Details |
|---------|---------|
| Full dark mode | Night variants across drawables, colors, layouts and animations |
| ColorOS design system | Built on the `coui` component library: motion curves, dialog transitions, Lottie loaders |
| Settings, About, Privacy | Dedicated screens, including in-app open-source notices (`OpenSourceNoticeActivity`) |
| In-app feedback | Bundled customer feedback SDK with its own multi-process activity |

## 🏗 Architecture

There is no source tree in this repository: the project ships one artifact, [Calculator.apk](Calculator.apk).
Its runtime architecture, decoded from `AndroidManifest.xml`, looks like this:

```text
                       ┌──────────────────────────────────────┐
                       │        DispatcherActivity            │
                       │  routes coloros.intent.action.* URIs  │
                       └───────┬──────────────────────────────┘
                               │
          ┌────────────────────┼───────────────────────────┐
          ▼                    ▼                           ▼
 ┌─────────────────┐  ┌──────────────────┐        ┌───────────────────┐
 │  Calculator     │  │  Converters      │        │  Finance          │
 │  (main + sci)   │  │  8 unit screens  │        │  Mortgage +       │
 │  MiniActivity   │  │  ExchangeRate    │        │  RateDialog       │
 └───────┬─────────┘  └────────┬─────────┘        └─────────┬─────────┘
         │                     │                            │
         ▼                     ▼                            ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │                    Content providers (data bus)                     │
 │   CalculatorProvider · UnitConvertProvider · CurrencyConvertProvider│
 └───────────────────────────┬─────────────────────────────────────────┘
                             │
        ┌────────────────────┼──────────────────────────┐
        ▼                    ▼                          ▼
 ┌──────────────┐   ┌─────────────────┐      ┌──────────────────────┐
 │ FloatWindow  │   │ Lock screen     │      │ System surfaces      │
 │ Service      │   │ card providers  │      │ settings search,     │
 │ (mini mode)  │   │ + components.rpk│      │ sidebar, feedback    │
 └──────────────┘   └─────────────────┘      └──────────────────────┘
```

A few things worth calling out:

- **One dispatcher, many entries.** `DispatcherActivity` is the exported hub: every
  `coloros.intent.action.*` (calculator, exchange rate, each unit family) lands there and is
  routed to the right screen, so external apps and system surfaces need to know only one entry point.
- **Providers as the shared data layer.** Calculation state, unit data and currency rates each get
  their own provider, which is how the main app, the mini window, and the lock screen cards all read
  the same data across processes.
- **Feature flags without recompiles.** Loan-type screens are declared as XML preference fragments
  (`res/xml/fragment_mortgage_convert_*.xml`), so the mortgage flow is data-driven.
- **Telemetry is pluggable.** OPPO's `nearx track`, statistics DCS, `epona` and `tingle` providers
  ride along behind custom permissions; on a non-OPPO device they simply stay dormant.

## 📁 Inside the APK

```text
Calculator.apk                         16.5 MB, ~8,900 classes across 2 dex files, no native code
├── AndroidManifest.xml                26 activities, 4 services, 20 providers, receivers
├── classes.dex                        main app (8,703 classes)
├── classes2.dex                       auxiliary classes (174)
├── res/
│   ├── layout/          (408)         screens, calculator keypad, dialogs
│   ├── drawable/        (1,205)       vector icons and gradients
│   ├── color/           (257)         color state lists, light and night variants
│   ├── anim/ + animator/(142)         coui motion curves, dialog transitions
│   ├── raw/             (106)         key sounds and Chinese voice clips (.ogg/.wav)
│   └── xml/                           mortgage flows, settings, lock screen cards, network config
├── assets/
│   ├── images/ + images_night/        🌙 dark-background animation frame sequences
│   ├── coui_lottie_*.json             Lottie loading animations, light and night
│   ├── category_translation.json      unit converter categories
│   ├── area_translation.json          unit conversion factors
│   ├── components.rpk                 lock screen card package
│   └── dexopt/baseline.prof           startup baseline profile
├── lib/                               (none: pure-DEX build, extractNativeLibs=false)
└── META-INF/                          v1 signature + okhttp/jersey service registrations
```

## 🛠 Tech stack

| Layer | Technology |
|-------|------------|
| Language | Kotlin and Java, compiled to 2 DEX files |
| UI toolkit | Android View system with OPPO's `coui` component library |
| Persistence | Room (`androidx.room` with multi-process invalidation) |
| Networking | OkHttp 3 (used by the exchange-rate and feedback features) |
| Animation | Lottie compositions and frame-sequence animations, light and night |
| App startup | `androidx.startup`, `ProfileInstaller` baseline profile |
| Platform | minSdk 30, targetSdk 36, compileSdk 36 |
| Signing | v1 (JAR) + v2/v3 APK Signature Scheme |
| OPPO platform libs | pantanal cards, epona, tingle IPC, nearx track, statistics DCS |

## 🚀 Getting started

### Prerequisites

- An Android device or emulator running **Android 11 (API 30) or newer**
- [ADB](https://developer.android.com/tools/adb) (part of Android platform-tools) for installation

### Install

```bash
# from a connected device
adb install Calculator.apk

# or if an older version with the same package is present
adb install -r Calculator.apk
```

Then launch it:

```bash
adb shell am start -n com.coloros.calculator/com.android.calculator2.Calculator
```

> **Note:** on phones where a stock calculator is preinstalled as a system app (OPPO, Samsung, etc.),
> installing this build requires removing or disabling the stock one first, or installing for a
> different user profile. Behavior here is device dependent.

### First-run notes

- The app requests `POST_NOTIFICATIONS` on first launch (Android 13+ runtime permission).
- Exchange-rate data refreshes over the network; unit conversion and the calculator itself work offline.
- On Samsung devices, the added `isSamsungDevice` check routes initialization down the corrected path automatically; there is no toggle.

## 📦 Build and signing

This repository ships a **prebuilt APK**; there is no Gradle project or build script here.
What is verifiable about the artifact:

| Property | Value |
|----------|-------|
| Package | `com.coloros.calculator` (original package `com.android.calculator2`) |
| Version | 17.0.0 (versionCode `17000000`) |
| Packaged | 2026-09-25 (zip entry timestamps) |
| Signatures | v1 `META-INF/ANDROIDD.SF` + v2/v3 APK Signing Block |
| Splits | None: single universal APK, all densities bundled |

If you unpack and modify the APK yourself, the usual toolchain applies
(`apktool` to decode/build, `zipalign`, then `apksigner` so the v2/v3 block is regenerated).
Re-signing is mandatory: resource or DEX changes invalidate existing signatures.

## 🔌 Integration surface

The app is callable from other apps and from system surfaces through intent actions and providers.

<details>
<summary><strong>Intent actions</strong> (click to expand)</summary>

| Action | Purpose |
|--------|---------|
| `coloros.intent.action.CALCULATOR` | Open the calculator |
| `coloros.intent.action.EXCHANGE_RATE` | Open currency conversion |
| `coloros.intent.action.LENGTH_UNIT` | Open length converter |
| `coloros.intent.action.AREA_UNIT` | Open area converter |
| `coloros.intent.action.VOLUME_UNIT` | Open volume converter |
| `coloros.intent.action.WEIGHT_UNIT` | Open weight converter |
| `coloros.intent.action.TEMPERATURE_UNIT` | Open temperature converter |
| `coloros.intent.action.POWER_UNIT` | Open power converter |
| `coloros.intent.action.PRESSURE_UNIT` | Open pressure converter |
| `coloros.intent.action.SPEED_UNIT` | Open speed converter |
| `oplus.intent.action.CALCULATOR` | Alternate calculator deep link |
| `oplus.intent.action.EXCHANGE_RATE` | Alternate exchange-rate deep link |
| `oplus.intent.action.CALCULATOR_SETTING` | Open calculator settings |
| `oplus.intent.action.MINI_LAUNCHER_MAIN` | Open the floating mini window |
| `com.oplus.calculator.EVALUATE_EXPRESSION` | Ask the calculator to evaluate an expression (smart sidebar) |

Source of truth: `AndroidManifest.xml` inside [Calculator.apk](Calculator.apk).

</details>

<details>
<summary><strong>Content providers</strong> (click to expand)</summary>

| Authority | Provider | Purpose |
|-----------|----------|---------|
| `com.android.calculator2.calculator` | `CalculatorProvider` | Calculation history and state |
| `com.android.calculator2.unit` | `UnitConvertProvider` | Unit conversion data |
| `com.android.calculator2.currency` | `CurrencyConvertProvider` | Exchange-rate data |
| `com.android.calculator2.card` | Lock screen card providers (domestic + export) | Lock screen card content |
| `com.coloros.calculator` | `SettingsSearchIndexablesProvider` | Settings search index |
| `com.coloros.calculator.androidx-startup` | `InitializationProvider` | androidx startup phases |
| `com.coloros.calculator.Track.*` | nearx track providers | Telemetry storage (OPPO devices) |
| `com.coloros.calculator.tingle` / `.epona` | OPPO IPC providers | Platform IPC (OPPO devices) |

</details>

<details>
<summary><strong>Permissions</strong> (click to expand)</summary>

| Permission | Why |
|------------|-----|
| `INTERNET`, `ACCESS_NETWORK_STATE` | Exchange-rate refresh, feedback upload |
| `POST_NOTIFICATIONS` | Notifications on Android 13+ |
| `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_DATA_SYNC` | Mini window and sync work |
| `SYSTEM_ALERT_WINDOW` | Floating calculator window |
| `VIBRATE` | Keypad haptics |
| `WRITE_SETTINGS`, `MODIFY_AUDIO_SETTINGS` | Settings integration, key sounds |
| `QUERY_ALL_PACKAGES` + OPLUS `permission.safe.*` | Platform integrations, mostly dormant off-OPPO |
| `oplus.bracketspace.permission.INSERT_PERMISSION` | Smart sidebar card |

</details>

## 💡 Why this exists

OPPO's ColorOS Calculator is one of the most complete calculator apps on Android:
scientific mode, eight unit families, live currency, a mortgage planner, a floating window,
and lock screen cards, all in a 16.5 MB package with no ads.

It is also locked to OPPO's own devices. This project exists to unlock it:
**it would not run on Samsung phones**, so this build adds Samsung device detection to fix
initialization, layers in a custom dark-background animation, re-signs the APK, and packages
the result so anyone on Android 11 or newer can sideload it.

> **Disclaimer:** ColorOS Calculator is proprietary OPPO software. This repository redistributes a
> modified build for personal use and study. No license is included; all rights to the original app
> remain with OPPO/OnePlus.

---

<div align="center">
<sub>One APK, every calculator. 🧮</sub>
</div>
