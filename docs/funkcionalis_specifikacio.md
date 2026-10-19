# FUNKCIONÁLIS SPECIFIKÁCIÓ ÉS DOKUMENTÁCIÓ

## 1. A Rendszer Célja és Áttekintése
A **PhotoMatch** egy webes és mobilos alapú fotós portfólió platform, amelynek célja, hogy összekösse a professzionális fotósokat a potenciális megrendelőkkel (magánszemélyekkel és vállalkozásokkal).

Az alkalmazás segítségével a fotósok bemutathatják vizuális munkáikat, szakterületek szerint kategorizálhatják képeiket (pl. esküvő, portré, rendezvény, autó), a megrendelők pedig összetett szűrőrendszeren keresztül böngészhetnek a szakemberek között, és közvetlen ajánlatot küldhetnek nekik.

---

## 2. Funkcionális Követelmények

### 2.1. Autentikáció, Profil- és Cégkezelés (`roles`, `accounts`, `profiles`, `companies`)
* **Egységes autentikációs réteg (`accounts` és `roles` táblák):**
  * Minden felhasználó (magánszemély ügyfél, céges ügyfél, fotós, adminisztrátor) közös hitelesítési táblán (`accounts`) keresztül jelentkezik be e-mail cím (`email`) és titkosított jelszó (`password_hash`) segítségével.
  * Szerepkör-alapú hozzáférés-vezérlés (`role_id` -> `roles`): Admin, Fotós, Ügyfél.
* **Magánszemély és Fotós profilok (`profiles` tábla):**
  * Az `accounts` táblához 1:1 kapcsolattal (`account_id`) illeszkedő személyes és szakmai adatok: név (`name`), telefonszám (`phone`), város (`city`), bemutatkozás (`bio`) és profilkép (`profile_image_url`).
* **Céges megrendelői profilok (`companies` tábla):**
  * Jogi személyként regisztráló megrendelők esetén az `accounts` táblához 1:1 kapcsolattal (`account_id`) rögzített vállalati adatok:
    * Cégnév (`company_name`)
    * Adószám (`tax_number`)
    * Cégjegyzékszám (`registration_number`)
    * Számlázási cím (`billing_address`)
    * Weboldal (`website`) és céges telefonszám (`phone`)

### 2.2. Portfóliókezelés és EXIF Metaadat-kinyerés (`photos`, `photo_metadata`, `categories`, `photo_categories`)
* **Képfeltöltés és bélyegkép-kezelés (`photos` tábla):**
  * Feltöltéskor a rendszer rögzíti a teljes méretű kép URL-címét (`image_url`), és automatikusan előállít egy kisebb méretű bélyegképet (`thumbnail_url`) a gyors betöltődés érdekében.
  * Cím (`title`) és leírás (`description`) megadása, valamint a megtekintések automatikus számlálása (`view_count`).
* **Automatizált EXIF metaadat-kinyerés (`photo_metadata` tábla):**
  * A kép feltöltése során a szerveroldal automatikusan kiolvassa és külön táblában (1:1 kapcsolatban) elmenti a kép fizikai EXIF adatait:
    * Kamera márka és modell (`camera_make`, `camera_model`)
    * Objektív típusa (`lens_model`)
    * Fókusztávolság (`focal_length`)
    * Rekeszérték (`aperture`, pl. f/1.8)
    * Záridő (`shutter_speed`, pl. 1/1000s)
    * ISO érzékenység (`iso`)
* **Metaadatok láthatóságának vezérlése (`show_metadata`):**
  * A fotós képenként beállíthatja (`boolean`), hogy a látogatók számára megjelenjenek-e a rögzített EXIF adatok a publikus felületen.
* **Többes kategória-besorolás (`categories`, `photo_categories`):**
  * Egy-egy fotó több kategóriába is besorolható (N:M kapcsolat).

### 2.3. Ajánlatkérési Rendszer (`inquiries`)
* **Ajánlatkérő űrlap:**
  * A megrendelő (`client_id` -> `accounts.id`) közvetlen ajánlatot küldhet a kiválasztott fotósnak (`photographer_id` -> `accounts.id`).
  * Rögzítendő adatok: Esemény típusa (`event_type`), dátuma (`event_date`), helyszíne (`location`), részletes üzenet (`message`).
* **Státuszkezelés (`status`):**
  * Az ajánlatkérés életciklusa nyomon követhető: `pending` (függőben), `accepted` (elfogadva), `rejected` (elutasítva), `completed` (teljesítve).

### 2.4. Értékelési és Véleményezési Rendszer (`reviews`)
* **Hitelesített értékelések:**
  * Kizárólag lezárt, sikeres ajánlatkéréshez (`inquiry_id`, 1:1 kapcsolat) kapcsolódóan hagyhat a megrendelő értékelést.
  * Adatok: Csillagos értékelés (`rating`, 1–5 skála) és szöveges vélemény (`comment`).

### 2.5. Archiválás és Puha Törlés (Soft Delete & Archiving)
Az adatbázis-integritás megőrzése és az adatvesztés elkerülése érdekében az alkalmazás puha törlést és archiválást alkalmaz végleges fizikai törlés helyett:
* **Ajánlatkérések archiválása (`archived_at`):** A lezárt vagy elutasított ajánlatkérések az "Archívum" mappába helyezhetők, így nem terhelik az aktív vezérlőpultot, de bármikor visszakereshetők.
* **Puha törlés (`deleted_at`):**
  * **Fotók törlése:** A fotó kikerül a publikus galériából (`deleted_at` időbélyeget kap), de a statisztikák megmaradnak.
  * **Fiókok és rekordok törlése:** Fiók (`accounts`, `profiles`, `companies`), ajánlatkérés (`inquiries`) vagy értékelés (`reviews`) törlése esetén a rekord inaktív státuszba kerül, megőrizve a korábbi tranzakciók és értékelések referenciális integritását.
