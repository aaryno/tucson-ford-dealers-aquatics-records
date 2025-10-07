# Tucson Ford Dealers Aquatics - Team Records

---

## ⚠️ **DRAFT RECORDS - NOT OFFICIAL** ⚠️

**THIS REPOSITORY CONTAINS DRAFT RECORDS FOR REVIEW AND VERIFICATION PURPOSES ONLY.**

These records are **NOT official** and are subject to change. Official team records are maintained by TFDA coaching staff. This repository is used for data analysis, verification, and coach review.

**DO NOT CITE THESE AS OFFICIAL RECORDS WITHOUT COACH APPROVAL.**

---

**Official Team Website:** https://www.gomotionapp.com/team/astfda/page/home

This repository contains draft copies of team records for Tucson Ford Dealers Aquatics (TFDA), a competitive swim team based in Tucson, Arizona.

## About Tucson Ford Dealers Aquatics

Tucson Ford Dealers Aquatics has been a cornerstone of competitive swimming in Arizona for over 27 years, with a rich history dating back to 1998. The team has produced numerous elite swimmers who have competed at the highest levels of the sport.

### Notable Accomplishments

**Olympic Champions:**
- **Matt Grevers** - Multiple Olympic gold medalist (2008, 2012) in backstroke events
- **Amanda Beard** - Seven-time Olympic medalist (1996, 2000, 2004, 2012) in breaststroke and individual medley

**International Elite Swimmers:**
- **Darian Townsend** - South African Olympic medalist and World Champion
- **Mike Alexandrov** - Bulgarian national record holder and World Championships competitor
- **Alyssa Anderson** - USA Swimming National Team member

**Program Highlights:**
- **900+ international swims** across Olympics, World Championships, and World Junior Championships
- **184 international medals** (66 gold, 63 silver, 55 bronze)
- **31 world, Olympic, and American records** set by Ford swimmers
- **27 years of continuous competition** (1998-2025)
- **258,000+ recorded swims** in USA Swimming database
- **186 unique swimmers** in team history

## Quick Links

- **USA Swimming Database:** https://usaswimming.sisense.com
- **SwimCloud Team Page:** https://www.swimcloud.com/team/8136/
- **Arizona Swimming (LSC):** https://www.azlsc.org/

## About This Repository

This repository publishes **draft copies** of team records generated from USA Swimming's official database and World Aquatics international competition results. These records are intended for:

- Team record verification
- Historical analysis
- Swimmer recognition
- Data transparency

**Note:** Official team records are maintained by TFDA coaching staff. This repository provides data-driven analysis to support record-keeping efforts.

### Annual Season Summaries

**NEW:** Season-by-season breakdowns of records broken:
- **[2024-2025 Season Summary](records/annual-summary-2025.md)** - 32 records broken, including Caden Castillo's incredible 13-record season
- **[2023-2024 Season Summary](records/annual-summary-2024.md)** - 4 records broken across SCY events

## Repository Structure

```
├── README.md                          # This file
├── METHODOLOGY.md                     # Complete technical methodology
└── records/
    ├── annual-summary-2024.md        # 2023-2024 Season records summary
    ├── annual-summary-2025.md        # 2024-2025 Season records summary
    ├── scy/                          # Short Course Yards records
│   │   ├── boys_records.md           # Boys individual records (all age groups)
│   │   ├── girls_records.md          # Girls individual records (all age groups)
│   │   ├── boys_relays.md            # Boys relay records
│   │   ├── girls_relays.md           # Girls relay records
│   │   ├── boys_top10.md             # Boys top 10 all-time
│   │   └── girls_top10.md            # Girls top 10 all-time
│   └── lcm/                          # Long Course Meters records
│       ├── boys_records.md           # Boys individual records
│       ├── girls_records.md          # Girls individual records
│       ├── boys_relays.md            # Boys relay records
│       ├── girls_relays.md           # Girls relay records
│       ├── boys_top10.md             # Boys top 10 all-time
│       └── girls_top10.md            # Girls top 10 all-time
└── METHODOLOGY.md                    # Complete data methodology
```

## Data Sources

All records are generated from official competitive swimming databases:

### 1. USA Swimming Official Times Database

**Primary Source:** https://usaswimming.sisense.com/api/datasources/USA%20Swimming%20Times%20Elasticube/jaql

- Official sanctioned meet results from USA Swimming
- **Coverage:** 1998-2025 (27 years)
- **Swims recorded:** 258,475 total swims
- **Courses:** Short Course Yards (SCY), Long Course Meters (LCM), Short Course Meters (SCM)
- **Data points:** Swim times, swimmer names, ages, meet names, dates, team affiliations
- **Authority:** Official USA Swimming database (authoritative for all records)

### 2. World Aquatics International Results

**Source:** World Aquatics API (formerly FINA)

- **Coverage:** Olympics, World Championships, World Junior Championships
- **Ford swimmers tracked:** 5 elite swimmers with international profiles
- **International swims:** 900 total swims
- **Medals:** 184 (66 gold, 63 silver, 55 bronze)
- **Records:** 31 world, Olympic, and American records
- **Authority:** Official international competition results

### 3. SwimCloud (Cross-Reference Only)

**Source:** https://www.swimcloud.com/team/8136/

- Third-party aggregator of swim results
- Used for data validation and verification
- **Not used for official records** (reference only)

## Methodology

### Record Generation Process

Records are generated through a multi-stage automated process:

#### Stage 1: Data Collection
1. **USA Swimming API queries** for all TFDA team swims (1998-2025)
2. **Individual swimmer career downloads** for unattached swim classification
3. **World Aquatics API queries** for international competition results

#### Stage 2: Swim Classification
All swims are classified into one of four categories:

1. **Official Ford Team Swims** ✅
   - Swims officially recorded with "Tucson Ford Dealers Aquatics"
   - Swims recorded with "Jim Click Team Elite, Inc" (Ford affiliate/sponsor)
   - **These always count toward records**

2. **Probationary Unattached Swims** (‡)
   - Unattached swims BEFORE first Ford swim
   - Occurred during club transfer probationary period
   - Swimmer joined Ford within 1 year after swim
   - **Included in records** with (‡) indicator

3. **Ford-Bounded Unattached Swims** (†)
   - Unattached swims AFTER first Ford swim
   - Includes: college meets, international competitions, time trials
   - Swimmer had ≥1 official Ford swim in same calendar year
   - **Included in records** with (†) indicator

4. **International Competition Swims** (◊)
   - Olympics, World Championships, World Junior Championships
   - Swimmer had ≥1 official Ford swim in same calendar year
   - Uses "Strict Temporal Boundaries" rule
   - **Included in records** with (◊) indicator

#### Stage 3: Ford Affiliation Analysis

**Strict Temporal Boundaries Rule:**

For unattached and international swims to count as Ford records, the swimmer must have competed for Ford in the **same calendar year** as the swim.

**Implementation:**
- Each swimmer's complete career is analyzed
- All USA Swimming swims are scanned for Ford team affiliation
- Unattached/international swims are matched to Ford membership years
- Swims qualify if `ford_year = TRUE` (≥1 Ford swim that year)

**Examples:**

✅ **QUALIFIES:** Matt Grevers, 100m Backstroke, 2012 Olympics
- Date: July 30, 2012
- Ford swims in 2012: Yes (4 Ford swims in 2012)
- **Status:** During-Ford-Year ✅
- **Counts as Ford record** with (◊) indicator

❌ **DISQUALIFIED:** Amanda Beard, 200m Breaststroke, 2008 Olympics  
- Date: August 13, 2008
- Ford swims in 2008: No (0 Ford swims in 2008)
- Last Ford swim: June 16, 2005 (1,154 days before)
- Next Ford swim: June 10, 2010 (666 days after)
- **Status:** Between-Ford-Periods ❌
- **Does NOT count as Ford record**

✅ **QUALIFIES:** Probationary swimmer, 100 Free, January 2020
- Date: January 15, 2020
- First Ford swim: June 1, 2020
- Previous team: Sierra Marlins (last swim October 2019)
- **Status:** Transfer probationary period ✅
- **Counts as Ford record** with (‡) indicator

#### Stage 4: Record Compilation
1. All qualifying swims combined by event/age group/course
2. Sorted by swim time (fastest first)
3. Ranked to identify records and top 10 performances
4. Formatted with swimmer names, times, dates, meets, and indicators

### Age Groups

Records are maintained for the following age groups:

- **10 & Under** (10U)
- **11-12**
- **13-14**
- **15-16**
- **17-18**
- **Open** (all ages, including 19+)

### Events

#### Short Course Yards (SCY)
**Freestyle:** 50, 100, 200, 500, 1000, 1650  
**Backstroke:** 50, 100, 200  
**Breaststroke:** 50, 100, 200  
**Butterfly:** 50, 100, 200  
**Individual Medley:** 100, 200, 400  
**Relays:** 200 Free, 400 Free, 800 Free, 200 Medley, 400 Medley

#### Long Course Meters (LCM)
**Freestyle:** 50, 100, 200, 400, 800, 1500  
**Backstroke:** 50, 100, 200  
**Breaststroke:** 50, 100, 200  
**Butterfly:** 50, 100, 200  
**Individual Medley:** 200, 400  
**Relays:** 200 Free, 400 Free, 800 Free, 200 Medley, 400 Medley

## Swim Indicators

Records include the following indicators to show data source:

- **(no indicator)** = Official Ford team swim (most common)
- **(‡)** = Probationary period unattached swim (before joining Ford)
- **(†)** = Ford-bounded unattached swim (college, time trial, etc.)
- **(◊)** = International competition (Olympics, Worlds, World Juniors)

All indicated swims have been verified to occur during a calendar year when the swimmer had ≥1 official Ford team swim.

## Statistics

### Data Coverage
- **Total swims analyzed:** 282,086
- **Official Ford team swims:** 237,132 (84%)
- **Probationary swims:** 4,044 (1.4%)
- **Ford-bounded unattached:** 19,067 (6.8%)
- **International swims:** 594 (0.2%) [from 900 total, 66% qualified]
- **Swimmers tracked:** 186 unique swimmers
- **Years covered:** 1998-2025 (27 years)

### Records by Course
- **SCY:** 212 individual events, 30 relay events
- **LCM:** 253 individual events, 28 relay events
- **SCM:** 73 individual events (all unattached/international)

## Contributing

This repository is maintained by TFDA team analysts. If you notice any discrepancies or have additional information:

1. Check the [Issues](../../issues) page
2. Submit corrections with supporting documentation
3. Include meet name, date, and swimmer name for verification

## License

This data is derived from public USA Swimming and World Aquatics competition results. Records are published for team recognition and historical preservation.

## Updates

Records are updated periodically as new meet results are published and analyzed. Check the repository commit history for the latest updates.

---

**Last Updated:** October 2025  
**Contact:** Tucson Ford Dealers Aquatics  
**Website:** https://www.gomotionapp.com/team/astfda/page/home
