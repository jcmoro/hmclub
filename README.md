<div align="center">

<img src="logo.webp" alt="Hattrick Manager Club" width="200">

# Hattrick Manager Club

**Your club on your phone.**

Squad, skills, arena, economy, staff, training and fixtures in one place.<br>
Free, read-only, and nothing leaves your device.

![Tests](https://img.shields.io/badge/tests-567%20passing-2e7d32?style=flat)
![Architecture](https://img.shields.io/badge/architecture%20rules-34-2e7d32?style=flat)
![Kotlin](https://img.shields.io/badge/Kotlin-2.4-7F52FF?style=flat&logo=kotlin&logoColor=white)
![Android](https://img.shields.io/badge/Android-7.0%2B-3DDC84?style=flat&logo=android&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-arm64-000000?style=flat&logo=apple&logoColor=white)
![Read only](https://img.shields.io/badge/access-read--only-1565c0?style=flat)
![Price](https://img.shields.io/badge/price-free-6a1b9a?style=flat)

### [**→ jcmoro.github.io/hmclub**](https://jcmoro.github.io/hmclub/)

[Product page](https://jcmoro.github.io/hmclub/) ·
[Privacy policy](https://jcmoro.github.io/hmclub/privacy.html) ·
[en español](https://jcmoro.github.io/hmclub/privacidad.html) ·
[Issues](https://github.com/jcmoro/hmclub/issues)

</div>

---

This repository holds the product page and the privacy policy — static HTML, no dependencies, no
trackers. The app itself lives in a separate repository. What follows is how that app is built.

<div align="center">
<img src="shots/squad.png" width="200" alt="Squad list: every player with age, position and five skill tracks">
<img src="shots/player.png" width="200" alt="Player details: form, experience, last match and every skill">
<img src="shots/club.png" width="200" alt="Club: league standing with your team marked, and the fixtures ahead">
<img src="shots/matches.png" width="200" alt="Matches: every fixture with its competition, date and result">
</div>

<div align="center">
<sub>
Squad · Player · Club · Matches. Light and dark, following your phone or fixed from the app.
</sub>
</div>

## Built once, for two phones

<div align="center">

![Kotlin](https://img.shields.io/badge/Kotlin-2.4-7F52FF?style=for-the-badge&logo=kotlin&logoColor=white)
![Multiplatform](https://img.shields.io/badge/Multiplatform-Android%20%2B%20iOS-7F52FF?style=for-the-badge)
![Compose](https://img.shields.io/badge/Jetpack%20Compose-Material%203-4285F4?style=for-the-badge&logo=jetpackcompose&logoColor=white)

![Ktor](https://img.shields.io/badge/Ktor-3.4-087CFA?style=flat-square&logo=ktor&logoColor=white)
![SQLDelight](https://img.shields.io/badge/SQLDelight-2.3-005571?style=flat-square&logo=sqlite&logoColor=white)
![Coroutines](https://img.shields.io/badge/Coroutines-1.10-7F52FF?style=flat-square)
![Serialization](https://img.shields.io/badge/kotlinx-serialization-7F52FF?style=flat-square)
![Gradle](https://img.shields.io/badge/Gradle-9.7-02303A?style=flat-square&logo=gradle&logoColor=white)

![Konsist](https://img.shields.io/badge/Konsist-architecture%20rules-2e7d32?style=flat-square)
![ktlint](https://img.shields.io/badge/ktlint-1.6-2e7d32?style=flat-square)
![Warnings](https://img.shields.io/badge/warnings-as%20errors-2e7d32?style=flat-square)
![CI](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)

</div>

The domain, the CHPP client and the club reading are Kotlin Multiplatform. Only the screens are
written per platform, so a rule about what Hattrick means is written once and holds on both.

```mermaid
flowchart TD
    UI["androidApp<br>Compose Material 3"]
    CLUB["core:club<br>the club a manager sees"]
    CHPP["core:chpp<br>OAuth · transport · store"]
    GAME["core:game<br>money · time · XML"]
    HT(["Hattrick CHPP"])

    UI --> CLUB
    UI --> CHPP
    CLUB --> CHPP
    CLUB --> GAME
    CHPP --> GAME
    CHPP -->|"OAuth 1.0a, signed on the phone"| HT

    style HT fill:#587e1b,stroke:#3d5a13,color:#fff
    style UI fill:#e2f3c8,stroke:#587e1b
```

Each context keeps the same three layers, and the arrows only ever point inwards: the domain knows
nothing about Hattrick's XML, about SQL or about Android.

```mermaid
flowchart LR
    A["application<br>use cases"] --> D
    I["infrastructure<br>adapters: CHPP · SQLDelight · keystore"] -->|"implements its ports"| D["domain<br>model · rules · ports"]

    style D fill:#e2f3c8,stroke:#587e1b
```

That is not a diagram of good intentions. **34 architecture rules** run on every push and fail the
build with file and line if the domain reaches for a framework, if one context reaches into
another, or if a screen carries a hardcoded string.

## What the numbers say

| | |
|---|---|
| **567 tests** | on the JVM, on Android and on the iOS simulator |
| **34 architecture rules** | Konsist, run as tests, no exceptions list |
| **13 CHPP files** | each with its own version and its own cache lifetime |
| **5 CI jobs** | nothing merges unless every one of them is green |
| **0 warnings** | a Kotlin warning fails the build; ktlint guards the style |

## Decisions worth naming

**There is no server.** The app signs its own requests from your phone and keeps what it downloads
there. The access token is sealed with AES-256-GCM under a key that never leaves the Android
Keystore — or the iOS Keychain, device-only — and both are kept out of device backups.

**The words come from Hattrick.** Skill levels, positions, specialties, training types and moods
are read from Hattrick's own `translations` file. Nothing is hardcoded, so nothing drifts and
everything reads right in any language.

**Unrevealed is not zero.** A skill Hattrick does not report is drawn differently from a skill at
level zero, because they are not the same thing. A test fixes that distinction.

**What gets stored is the raw XML**, not the mapped model. When a mapping turns out to be wrong,
the fix re-derives the whole history instead of leaving bad numbers frozen in a database.

**Mappings are written against captured documents**, never against what the documentation says.
Three bugs in a previous version came from a fixture and the code agreeing on the same wrong
assumption; real answers from Hattrick are what the tests read now.

**Failures report themselves.** If the app crashes, or Hattrick sends something it cannot read, it
opens an issue in this repository on its own — with the class, the line and the phone model, and
with nothing about your club, your players or your token.

---

<div align="center">
<sub>
A CHPP product, built by one Hattrick manager.<br>
Not affiliated with, endorsed by, or operated by Hattrick Holdings Limited.<br>
Hattrick and the Hattrick logo are theirs.
</sub>
</div>
