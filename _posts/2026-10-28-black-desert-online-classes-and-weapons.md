---
title: "Black Desert Online Classes, Weapons and What Comes Next"
date: 2026-10-28
layout: splash
categories:
  - blog
tags:
  - bdo

excerpt: "How the client numbers its classes and weapons - and what that says about the next ones"
header:
  overlay_color: "#252a34"
  # overlay_image: assets/images/bdo_classes_weapons/header.webp
author_profile: false
toc: true
toc_label: "Classes & Weapons"
---
Hey there, its been a hot minute! Same as last time, this writeup documents a little rabbit hole of mine - personal notes and observations I wanted to share. The below writeup is quite nerdy and probably not really interesting for most people who'll stumble accross it, but I figured I'd just write it down either way.

One note before we start: While I did write down a lot of notes etc. I let AI help me write this text since I am quite the scatterbrain often and mix things up - so if some parts read weird, blame AI xD

Nothing in here is official, and a good part of the last sections is educated guessing. Keep that in mind before you go and announce new classes. :D

# Backstory
While working on my own tooling for Black Desert Online's (BDO) files, I kept running into the same question: how does the game actually keep track of its classes?
Every class has a model folder, an action chart, a slot in a class table, a set of weapons, starter items, customization data... and all of these are numbered. But the numbers never quite line up with each other, and they definitely don't line up with the release order.

So I sat down and lined them all up. I compared client builds from several regions, from the oldest one I have with this data (late 2024) up to the newest one (October 2026), and followed every class through them.
It turns out the numbers tell quite a story: renamed classes, classes that were planned and then dropped, lore pairs that were designed together but released years apart, and a pretty clear picture of how a new class makes its way into the files.

# How BDO numbers its classes
There are four numbers per class, and each one is its own little clock:

| Number | Where it lives | What it tells you |
|---|---|---|
| **Code number** | the class's model folder, e.g. `1_phm` | the order in which the class's code was created. A renamed class keeps its number |
| **Class slot** | the class table | the id the game uses at runtime |
| **Gear number** | the weapon and armor folders, e.g. `29_doubleax` | one counter shared by weapons *and* armor: the order in which weapon types were created |
| **Item keys** | the starter weapons | the order in which the items were made |

## The class codes
The codes are short and, at least for the old classes, actually readable. They start with `p` (player character), then a letter for the race or theme, then the gender:

- `phm` / `phw`: **h**uman **m**an and **w**oman - Warrior and Sorceress
- `pew` / `pem`: **e**lf - Ranger and, years later, Archer (internally simply "elfman")
- `pgm` / `pgw`: **g**iant - Berserker (internally still called "Giant") and Guardian ("giantwomen")
- `pnm` / `pnw`: **n**inja - Ninja and Kunoichi
- `pcm` / `pcw`: **c**ombattant - Striker and Mystic

From around Wukong on, the codes stop following that pattern (`pjkd`, `prsa`, `pwge`, `pgms`...), so the letters don't tell you much anymore.

## Class slots come in blocks of four
This was my favorite find. The class table was laid out in blocks of four: every original class got one slot, plus **three spare "job" slots** right after it. Back in 2014 the plan was apparently a job-change system - the old names of those slots read like "Warrior job-change 1 (temporary)" or "Warrior job 3 (unused)".

That system never came, but the slots didn't go to waste. Later classes simply moved into whatever spare slot was free:

| Block | Base class | Spare slots, as filled later |
|---|---|---|
| 0-3 | Warrior | Hashashin, Sage, Wukong |
| 4-7 | Ranger | Guardian, Scholar, Drakania |
| 8-11 | Sorceress | Nova, Corsair, Lahn |
| 12-15 | Berserker | *PBM (never released)*, *Ain (unused)*, Maegu |
| 16-19 | Tamer | Shai, *Shai male (unused)*, Striker |
| 20-23 | Musa | Maehwa, *Temporary Maehwa (unused)*, Mystic |
| 24-27 | Valkyrie | Kunoichi, Ninja, Dark Knight |
| 28-31 | Wizard | Archer, Woosa, Witch |

After that, the newer classes just get appended: Seraph (32), Dosa (33), Deadeye (34) and Agent (35).

Funnily enough, the job slots mostly kept the gender of their base class: Warrior's three went to male classes, Ranger's and Sorceress's to female ones. Berserker's and Tamer's blocks break that rule (Maegu and Striker).

## All released classes in code order

| Code # | Code | Class | Slot |
|---|---|---|---|
| 1 | phm | Warrior | 0 |
| 2 | phw | Sorceress | 8 |
| 3 | pew | Ranger | 4 |
| 4 | pgm | Berserker | 12 |
| 5 | pbw | Tamer | 16 |
| 6 | pkm | Musa | 20 |
| 7 | pvw | Valkyrie | 24 |
| 8 | pwm / pww | Wizard / Witch | 28 / 31 |
| 9 | pem | Archer | 29 |
| 11 | pgw | Guardian | 5 |
| 12 | pkw | Maehwa | 21 |
| 13 | pnm / pnw | Ninja / Kunoichi | 26 / 25 |
| 14 | plw | Shai | 17 |
| 15 | pdw | Dark Knight | 27 |
| 16 | pcm / pcw | Striker / Mystic | 19 / 23 |
| 17 | psw | Lahn | 11 |
| 18 | pam | Hashashin | 1 |
| 20 | pjkd | Wukong | 3 |
| 21 | ppw | Nova | 9 |
| 23 | ppm | Sage | 2 |
| 24 | pfw | Corsair | 10 |
| 25 | pqw | Drakania | 7 |
| 27 | pkow | Maegu | 15 |
| 28 | pmyf | Woosa | 30 |
| 29 | pnyw | Scholar | 6 |
| 30 / 33 | pdkl | Seraph | 32 |
| 31 | prsa | Dosa | 33 |
| 32 | pwge | Deadeye | 34 |
| 34 | pgms | Agent | 35 |

The gaps are not random either. Number 10 was an old Kunoichi that got replaced, 19 is a developer test character, 22 the unused Maehwa remake and 26 the never-released PBM. More on those further down.

# Weapons
## Folder names tell you the era
Every weapon type has its own folder, and you can almost date a class by how its weapon folders are named:

- **Up to Guardian**, the names simply describe the weapon: `onehandsword`, `shield`, `doubleax`, `enchantarm`, `norigae`, `handcanon`, and my personal favorite, Shai's Florang living in a folder called `boommerang`.
- **Sage, Corsair and Drakania** got opaque three-letter codes like `cbe`, `kdy` or `bsg`.
- **From Wukong on**, there is a fixed rule: letters two and three of the class code plus `r` (main hand), `l` (off hand) or `a` (awakening). Dosa (`prsa`) has `rsr`, `rsl` and `rsa`, Deadeye (`pwge`) has `wgr`, `wgl`, `wga`, and so on.

## One counter for weapons and armor
The number in front of each folder comes from a single counter shared by weapons and armor. Armor sits in there too (upper body, gloves, shoes, cloak, underwear, earrings...), which leads to some fun collisions: number 38 is both the underwear slot and Sorceress's scythe, 84 and 85 are both Shai's weapons and Wukong's, and 98-100 are both Drakania's weapons and the remade Wizard weapons.

A few numbers never show up anywhere. Since the counter covers all equipment, my guess is that those are the accessories without a mesh (rings, belts, necklaces) - but that is just an inference.

## Gender pairs share their weapons
Male/female pairs of the same concept share their main weapon items: Warrior and Valkyrie, Musa and Maehwa, Striker and Mystic, Ninja and Kunoichi, Wizard and Witch. The newest pair is **Deadeye and Agent**: Agent's main weapon item is literally Deadeye's revolver. Each class still has its own models in its own folders, though.

# Renamed, reused and dropped
## Classes that started under another name
Codes do get renamed, and the number stays with them:

- **Wukong** was **PSM** until 2025. In the oldest build I have, its character-select entry is still called "도깨비 무사" - Dokkaebi warrior. By autumn 2025 it had become `pjkd`.
- **Seraph** started as **POSM**, became PDKL and was then released in Korea in December 2025.
- **Woosa** is `pmyf` today, but its character-select object and some customization effects still use the older name `pkyw`.

## The clocks don't always agree
For most classes the four numbers roughly agree. For three of them they really don't:

- **Wukong** has an old code (#20, right after Lahn and Hashashin) and old weapon numbers from the Nova/Shai era, but its items were created *after* Deadeye's.
- **Seraph**'s code and slot are older than Dosa's and Deadeye's, but its model folder and weapons come after theirs.
- **Maegu and Woosa**: the codes have Maegu first, the items have Woosa first.

## Things that never made it
A few entries in the files never became anything (or not yet):

- **Shai male**: a class slot built on Tamer's action chart, never used.
- **Ain**: an unused slot with some Warrior-based heads attached.
- **Temporary Maehwa**: a complete Maehwa remake model set - about 2000 armor files - that was never released. Maehwa's awakening weapon model lives in there, though.
- **The old Kunoichi**: replaced by today's Kunoichi; its slot later went to Woosa.
- **PBM**: a slot named "Giant (beast)", which fits a model folder with fur, armor and no weapons. Never released.
- **The Wizard remake**: the Wizard's items already point at a remade model set, while the Witch still uses the old one.

# Lore pairs in the data
Classes that were planned together really do show up next to each other in the files - they just weren't released together.

**Lahn and Wukong.** Lahn's code is `psw`, and her action chart is internally called "chinese". Wukong's first code was **PSM** - the same letters with an "m". So Wukong started as Lahn's male counterpart. His code number (#20) comes soon after Lahn's (#17), with only Hashashin and a developer test character in between, and his weapon numbers were assigned back then too. He was only built years later though (maybe because at the time BDO wasnt approved for a china release iirc): his slot is Warrior's last spare one, and his items were made after Deadeye's.

**Maegu and Woosa.** The sisters have consecutive codes (`pkow` and what used to be `pkyw`, so they started as `pk?w` twins), back-to-back weapon numbers and item keys made in one go. Their slots are far apart - 15 and 30 - so slots were clearly handed out by whatever was free, not by lore.

Bonus observation: the `k` might mark the Korean-themed classes - Musa `pkm`, Maehwa `pkw`, Maegu `pkow` and Woosa `pkyw`. Dosa came after the codes stopped following the letters, so he doesn't fit in.

# How a new class appears in the files
Following Wukong, Deadeye, Seraph and Agent through the builds, a new class goes through the same steps every time:

1. A generic placeholder character exists, and the next free class slot is named "PBM".
2. The placeholder gets the new class's code. It borrows the voice of an existing class (Deadeye's placeholder used Woosa's, Agent's used Dosa's) and sometimes even its starter weapons.
3. A character-select entry is added, marked "in development" and often sitting on a borrowed scene (several use Warrior's).
4. At release it gets its Korean name, its own voices, its own select movie and finally the flag that opens it.

For Agent, step 2 to release took about six months.

# What comes next
Now for the speculative part. As of the newest build I have:

| Code | What the files have | Status |
|---|---|---|
| **PNKF** ♀ | its own faces and hairstyles are already listed in a female placeholder slot | the newest code, and the only upcoming one with its own art referenced |
| **POEA** | a character-select entry marked "in development" since at least late 2024; the next free class slot points at it | its character is still the generic placeholder |
| **PBNA** | the same kind of select entry, the same age as POEA | in development; no class slot points at it yet |

- **PNKF looks like the next class.** It is the newest code, and it already has art. Its "f" also matches the female slot it sits in.
- **POEA and PBNA are older than you'd think.** Both have been "in development" since before Agent existed - and Agent overtook them.
- **Ten placeholder slots** were added at once: five male/female pairs that are plain copies of Dosa (male) and Woosa (female). That's reserved room, not ten classes. PNKF has already taken one of them.
- **Pace:** four classes went from placeholder to playable between late 2024 and mid 2026. That's two to three a year.

**A little trick to tell the gender early:** as mentioned above, a new class borrows the voice of an existing class - and it picks one of the planned gender. Deadeye's placeholder used Woosa's voice, Agent's used Dosa's. Interestingly, Seraph's placeholder POSM used Musa's voice and its code ended in "m", so Seraph most likely started out as a male concept.

**Weapons:** Agent's awakening is still missing; by the counter it will be number 129. A new class should get the next three numbers, 130-132, named after the rule from above - for PNKF that would be `nkr`, `nkl` and `nka`. What kind of weapon it will be is anyone's guess, since the names stopped describing the weapons a long time ago.

**And one more number:** released classes are currently at **20 female and 12 male**, and PNKF will be another female. If there's a gap anywhere, it's on the male side. The only male counterpart still hinted at in the files is an unused select entry that reads like a male Scholar.

# Bonus: Is Agent's awakening a scythe?
**Fair warning: this whole section is speculation.** Nothing in the game data confirms it - there is no awakening weapon, no awakening skill book and no model yet. This is only what the lore hints at.

Agent - Edward "Eddie" Canary, known as **Gannicus** in the lore - doesn't have an awakening yet, and players have been speculating that it will be a scythe. Since I had the lore texts lying around anyway, I went looking for hints. And there are quite a few:

- **The Gannicus poem.** In his story you come across a poem about the legendary sorcerer Gannicus. The Korean original reads "그의 낫끝은 불꽃을 뿜고, 날카롭고 검은 탄환이 적을 꿰뚫었도다" - "from the tip of his scythe sprang flame, and black bullets pierced his foes". So the legend describes a single weapon that is a scythe *and* fires bullets.
- **Gannicus's Hexround.** The quest item is described as a replica of the weapon Gannicus wielded "in place of a scythe" to unleash black magic.
- **His codename is Grim Reaper.** In Korean, his Succession is called "Codename: Grim Reaper" (암호명 그림리퍼), and Marni calls him that in his story cutscene. The English version shortened it to "Codename: G". It doesn't get much more scythe than the Grim Reaper. :D
- **The Sorceress connection.** The Sorceress's awakening weapon is Cartian's Scythe - and in Agent's story, Cartian tells him she studied his power and built the Sorceresses' magic on it. So in the lore, Tarif's dark magic and its scythes trace back to Gannicus.
- **Marni's tech, mistaken for sorcery.** Mr. Servant jokes that Marni's "masterwork" was mistaken for sorcery back then. So the legend's flaming scythe would be Marni technology, just like the hexround.

Now the other side, because I don't want to oversell this:

- **Some of the "reapers" exist only in English.** His skill "Hexround Reaper" is 마탄의 사수 in Korean - the Korean title of the opera *Der Freischütz*, "the marksman of the magic bullets". No reaper in there.
- **The Grim Reaper codename belongs to his current kit**, the revolvers. On its own it says nothing about the awakening.
- **"In place of a scythe"** could just as well mean he *didn't* use one, and the gun did the job instead.
- **Nothing in the class data yet.** Agent only has his Succession skill book so far, and his awakening weapon folder (number 129 from above) doesn't exist yet. Even when it shows up, the folder name will be a code, not "scythe".

**My take:** the lore is clearly setting something up, and a scythe that shoots would tie it all together nicely - the legend, the codename and the Sorceress's lineage. But until an awakening item or skill book shows up in the files, it stays exactly that: speculation. The first real evidence will be the Korean name of his awakening weapon - if it contains 낫 (scythe), we'll know.

# Conclusion
What started as "why don't these numbers match?" turned into a small archaeology project. It was fun to see old plans that never happened - like the job-change system - still shaping where new classes end up more than ten years later.

Once more: this is all read from client data. Codes have been renamed before, plans change, and none of this is an announcement. But next time a new code shows up in the files, at least I know where to look. :D
