# SleepReset — Workspace Plan

Single native SwiftUI iOS app (Health & Fitness). Base: fork of RestfulNight (MIT).
Positioning: sleep habits and relaxation. It is not a medical device, and it makes no diagnostic or treatment claims.

## 1. Module layout
```
SleepReset/
  App/                 SleepResetApp.swift, RootTabView, AppState, DI
  Core/                Models (SwiftData), Persistence, HealthKitService, Haptics, Analytics (local only)
  Features/
    Sleep/             From RestfulNight: sleep log, bedtime/wake schedule, history
    Charts/            Wraps SleepChartKit (SPM, MIT): stage timeline, weekly trends
    Breathe/           Adapted from iBreathe (MIT): 4-7-8, box, coherent breathing + Core Haptics
    Habits/            NEW native module (SwiftData): wind-down habits, streaks, reminders
    Sounds/            noctune seed: looping ambient sounds, sleep timer, fade-out
    Paywall/           StoreKit 2 subscription views
    Settings/          Permissions, notifications, export, privacy, licences screen
  Resources/           Audio (owned or CC0 only), assets, Localizable.xcstrings
  Tests/               Unit tests (models, streak logic) + UI smoke tests
```
SPM dependencies: `SleepChartKit` (MIT). Keep the dependency list minimal.

## 2. Screens
1. Onboarding (3 pages) → goal picker → HealthKit + notification permission primers
2. Tonight (home): bedtime countdown, wind-down checklist (habits), quick Breathe/Sounds
3. Sleep log / history (RestfulNight) + detail
4. Insights: SleepChartKit stage chart, 7- and 30-day trends, habit↔sleep correlation (premium)
5. Breathe: pattern list → session player (animation + haptics) → summary
6. Habits: list, add/edit habit, streak calendar
7. Sounds: library grid → player with timer and mix (premium mixes)
8. Paywall
9. Settings, privacy policy, third-party licences

## 3. Data model (SwiftData)
- `SleepSession` id, start, end, source (manual/healthKit), quality 1–5, notes, stages [SleepStageSample]
- `SleepStageSample` stage (awake/rem/core/deep), start, end
- `Habit` id, title, icon, schedule (daily/weekdays/custom), reminderTime?, targetPerDay, archived, createdAt
- `HabitLog` id, habit, date, count, completed
- `BreathSession` id, pattern, duration, date
- `SoundPreset` id, name, tracks [trackId: volume], isPremium
- `UserSettings` bedtimeGoal, wakeGoal, sleepGoalMinutes, hapticsOn, onboardingDone
Streaks are computed from HabitLog rather than stored. CloudKit sync is optional later (v1.1).

## 4. HealthKit
- Read: `sleepAnalysis`, `mindfulSession`, optional `heartRate` and `restingHeartRate` (insights only)
- Write: `sleepAnalysis` (manual logs), `mindfulSession` (breath sessions)
- Info.plist: NSHealthShareUsageDescription, NSHealthUpdateUsageDescription. Request permission in context, never at launch. The app must stay fully usable if access is denied.
- Health data stays on device. No ads and no third-party analytics SDK touch it (App Store rule 5.1.3).

## 5. Paywall placement
- Free: sleep log, basic charts (7 days), 3 habits, 2 breathing patterns, 5 sounds
- Premium (StoreKit 2, monthly and yearly with a 7-day trial): unlimited habits, 30/90-day insights + correlations, all patterns, sound mixer, export
- Show the paywall: at the end of onboarding (can be dismissed), when tapping a locked feature, and after the 3rd completed night. Never block the core sleep log.
- Required: Restore Purchases, terms and privacy links, clear pricing (rules 3.1.1/3.1.2)
- Prices are TBD. No spend is involved.

## 6. Licence obligations
- RestfulNight (MIT): keep its copyright and licence text (LICENSE in the fork + THIRD_PARTY_NOTICES.md)
- SleepChartKit (MIT): include its notice in THIRD_PARTY_NOTICES.md and the in-app Licences screen
- iBreathe (MIT): adapted code keeps a header comment crediting the source. Its licence notice goes in THIRD_PARTY_NOTICES.md
- Excluded: Brethap (GPL, copyleft) and Dreamflow (no licence)
- Audio: owned, generated, or CC0 only. Record each source in Resources/AUDIO-SOURCES.md
- Check each upstream LICENSE file at fork/import time and paste the exact copyright line into the notices file

## 7. 14-day build order
| Day | Work |
|---|---|
| 1 | Rename target/bundle to SleepReset, set up the folder layout above, CI build, add the SwiftData container |
| 2 | Migrate RestfulNight models to SwiftData SleepSession; get the sleep log working |
| 3 | HealthKitService: permission primer, read/write sleepAnalysis |
| 4 | Add SleepChartKit through SPM; Insights screen with 7-day chart |
| 5 | Habits: model, list, add/edit |
| 6 | Habits: logging, streak logic + unit tests, local notification reminders |
| 7 | Tonight home screen: countdown + wind-down checklist; buffer/QA |
| 8 | Breathe: port iBreathe patterns and animation, with credit header |
| 9 | Breathe: Core Haptics patterns (keep continuous events under 30 s), write mindfulSession |
| 10 | Sounds (noctune seed): AVAudioEngine looping, timer, fade, background audio |
| 11 | Onboarding + settings + Licences screen |
| 12 | StoreKit 2 paywall, entitlement gating, StoreKit config testing |
| 13 | Habit↔sleep correlation insight, accessibility, Dynamic Type, dark mode |
| 14 | QA pass (qa-checks repo), privacy manifest, screenshots, TestFlight build |
