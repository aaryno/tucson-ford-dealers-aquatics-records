# Records Generation Methodology

This document provides complete technical details on how Ford Aquatics team records are generated from official swimming competition databases.

## Table of Contents

1. [Data Collection Pipeline](#data-collection-pipeline)
2. [Swim Classification Rules](#swim-classification-rules)
3. [Ford Affiliation Determination](#ford-affiliation-determination)
4. [Record Compilation Process](#record-compilation-process)
5. [Quality Assurance](#quality-assurance)
6. [Technical Implementation](#technical-implementation)

---

## Data Collection Pipeline

### Stage 1: USA Swimming Database Collection

**API Endpoint:** `https://usaswimming.sisense.com/api/datasources/USA%20Swimming%20Times%20Elasticube/jaql`

**Collection Process:**

1. **Team-based queries** for all "Tucson Ford Dealers Aquatics" swims
   - Years: 1998-2025 (27 years)
   - Courses: SCY, LCM, SCM
   - Events: All individual and relay events
   - Result: 237,132 official Ford team swims

2. **Individual swimmer career downloads**
   - Query by PersonKey (USA Swimming unique ID)
   - All swims across entire career (all teams)
   - Purpose: Identify unattached swims for classification
   - Result: 186 complete swimmer profiles

3. **Data fields collected:**
   ```
   SwimTime, Name, Foreign, Age, Event, LSC, Team, Meet, SwimDate,
   MeetKey, TimeStandard, SwimEventKey, EventCompetitionCategoryKey,
   PersonKey, SortKey, UsasSwimTimeKey, Year
   ```

**Data Quality:**
- **Deduplication:** By `UsasSwimTimeKey` (unique swim identifier)
- **Validation:** All times verified as valid swim times
- **Completeness:** SwimDate field provides exact meet dates

### Stage 2: International Results Collection

**API Endpoint:** World Aquatics API (https://api.worldaquatics.com)

**Collection Process:**

1. **Swimmer identification:**
   - Search World Aquatics database for each Ford swimmer
   - Match by name, date of birth, nationality
   - Link USA Swimming `PersonKey` to World Aquatics `PersonId`
   - Result: 5 Ford swimmers with international profiles

2. **Competition results download:**
   - All Olympics, World Championships, World Juniors swims
   - Complete event details, times, rankings, medals
   - Date range: 1995-2024 (30 years)
   - Result: 900 international swims from 5 swimmers

3. **Data fields collected:**
   ```
   CompetitionName, CompetitionType, CompetitionCity, CompetitionCountry,
   Event, Phase, Time, Place, Age, Points, ClubName, Medal, RecordType,
   Notes, date, meet_name, event_name, category, gender
   ```

**Elite Swimmers with International Results:**
- Matt Grevers (PersonKey 620298): 281 swims, 36 medals
- Amanda Beard (PersonKey 3499680): 111 swims, 56 medals  
- Darian Townsend (PersonKey 1153369): 280 swims, 83 medals
- Mike Alexandrov (PersonKey 1190524): 196 swims, 9 medals
- Alyssa Anderson (PersonKey 1231116): 32 swims, 0 medals

---

## Swim Classification Rules

All swims are classified into one of four categories based on team affiliation and temporal analysis.

### Category 1: Official Ford Team Swims ✅

**Definition:** Swims officially recorded with Ford team affiliation in USA Swimming database.

**Criteria:**
- `Team = "Tucson Ford Dealers Aquatics"` OR
- `Team = "Jim Click Team Elite, Inc"` (Ford affiliate/sponsor name)

**Example:**
```
SwimTime: 48.32
Name: John Smith
Team: Tucson Ford Dealers Aquatics
Meet: 2024 Arizona SCY State Championships
Date: 03/15/2024
```

**Status:** ✅ Always qualifies as Ford record (no indicator needed)

### Category 2: Probationary Unattached Swims (‡)

**Definition:** Unattached swims that occurred BEFORE first Ford swim, during club transfer probationary period.

**Criteria (ALL must be true):**
1. `Team = "Unattached"`
2. Swim date is BEFORE first Ford swim date
3. Swimmer had previous club affiliation (not first-time registration)
4. Swimmer joined Ford within 365 days after this swim

**Classification Algorithm:**
```python
def is_probationary(swim_date, first_ford_date, previous_club_date):
    if swim_date < first_ford_date:
        if previous_club_date is not None:
            days_until_ford = (first_ford_date - swim_date).days
            if days_until_ford <= 365:
                return True
    return False
```

**Example:**
```
SwimTime: 50.12
Name: Jane Doe  
Team: Unattached
Meet: 2020 Winter Sectionals
Date: 01/15/2020
Previous team: Sierra Marlins (last swim 10/30/2019)
First Ford swim: 06/01/2020
Days until Ford: 138 days
Status: ✅ Probationary (‡)
```

**USA Swimming Rule:** Swimmers changing clubs must complete a 120-day probationary period (or until September 1 of competition year, whichever comes first). During this period, swimmers compete as "Unattached."

**Result:** 4,044 probationary swims from 81 swimmers

### Category 3: Ford-Bounded Unattached Swims (†)

**Definition:** Unattached swims that occurred AFTER first Ford swim, where swimmer remained Ford-affiliated.

**Criteria (ALL must be true):**
1. `Team = "Unattached"`
2. Swim date is AFTER first Ford swim date
3. Swimmer had ≥1 Ford swim in same calendar year
4. Not classified as international competition swim

**Classification Algorithm:**
```python
def is_ford_bounded(swim_date, ford_swims_by_year):
    swim_year = swim_date.year
    if swim_year in ford_swims_by_year:
        if len(ford_swims_by_year[swim_year]) > 0:
            return True
    return False
```

**Example:**
```
SwimTime: 47.89
Name: John Smith
Team: Unattached
Meet: 2024 NCAA Division II Championships
Date: 03/14/2024
Ford swims in 2024: Yes (12 Ford swims)
Nearest Ford swim: 02/28/2024 (14 days before)
Status: ✅ Ford-bounded (†)
Context: NCAA college meet
```

**Common Contexts:**
- **NCAA/College meets:** Swimmer competes for university team
- **Time trials:** Individual time verification swims
- **High school meets:** AIA championship swims
- **Olympic Trials:** USA Swimming national championships

**Result:** 19,067 Ford-bounded swims from 239 swimmers

### Category 4: International Competition Swims (◊)

**Definition:** Olympics, World Championships, or World Junior Championships swims by Ford swimmers.

**Criteria (ALL must be true):**
1. Competition type is Olympics, World Championships, or World Juniors
2. Swimmer is in Ford roster (PersonKey matches)
3. Swimmer had ≥1 Ford swim in same calendar year as international swim

**Classification Algorithm:**
```python
def is_ford_international(intl_swim_date, ford_swims_by_year):
    intl_year = intl_swim_date.year
    if intl_year in ford_swims_by_year:
        if len(ford_swims_by_year[intl_year]) > 0:
            return True
    return False
```

**Example:**
```
SwimTime: 52.16
Name: Matt Grevers
Team: USA (shown as "Unattached" in some databases)
Meet: 2012 Olympic Games
Event: 100m Backstroke
Date: 07/30/2012
Ford swims in 2012: Yes (4 Ford swims)
Status: ✅ During-Ford-Year (◊)
Result: Gold Medal 🥇
```

**Result:** 594 of 900 international swims qualified (66%)

---

## Ford Affiliation Determination

### Strict Temporal Boundaries Rule

**Definition:** A swim qualifies as a Ford record if the swimmer had **at least one official Ford team swim in the same calendar year** as the unattached or international swim.

**Implementation:**

1. **Build Ford membership timeline:**
   ```python
   ford_years = {}
   for swim in all_usa_swimming_swims:
       if swim.team == "Tucson Ford Dealers Aquatics":
           year = swim.date.year
           if year not in ford_years:
               ford_years[year] = []
           ford_years[year].append(swim.date)
   ```

2. **Check affiliation for each swim:**
   ```python
   def check_ford_affiliation(swim_date, ford_years):
       swim_year = swim_date.year
       return swim_year in ford_years and len(ford_years[swim_year]) > 0
   ```

3. **Assign affiliation status:**
   - `During-Ford-Year`: ✅ Qualifies (≥1 Ford swim in same year)
   - `Pre-Ford`: ❌ No Ford swims yet
   - `Post-Ford`: ❌ No more Ford swims after this year
   - `Between-Ford-Periods`: ❌ Gap in Ford membership

### Critical Case Study: Amanda Beard 2008 Olympics

This case demonstrates why temporal analysis is essential:

**Event:** Women's 200m Breaststroke, 2008 Beijing Olympics  
**Date:** August 13, 2008  
**Time:** 2:23.00  
**Place:** 7th place (semifinals)  

**Ford Affiliation Analysis:**
```
Ford swim history:
  1997-2005: Tucson Ford Dealers Aquatics (9 years)
  2008: NO Ford swims (competed for Trojan Swim Club)
  2010-2012: Tucson Ford Dealers Aquatics (3 years)

Temporal metrics:
  Last Ford swim before Olympics: June 16, 2005 (1,154 days before)
  Next Ford swim after Olympics: June 10, 2010 (666 days after)
  Ford swims in 2008: 0
  
Affiliation status: Between-Ford-Periods ❌

Decision: Does NOT count as Ford record
Reason: No Ford team affiliation in 2008
```

**Policy Rationale:** Amanda competed for Trojan Swim Club in 2008. Although she rejoined Ford in 2010, the 2008 Olympics swim does not qualify because she had NO Ford affiliation during that calendar year. This ensures fair attribution and prevents retroactive record claims.

**Amanda Beard's Olympic Record:**
- 1996 Olympics (age 14): ✅ NOT FORD (competed for Irvine Novaquatics before joining Ford)
- 2000 Olympics (age 18): ✅ COUNTS (Ford swimmer in 2000)
- 2004 Olympics (age 22): ✅ COUNTS (Ford swimmer in 2004)
- 2008 Olympics (age 26): ❌ DOES NOT COUNT (competed for Trojan Swim Club)
- 2012 Olympics (age 30): ✅ COUNTS (Ford swimmer in 2012)

**Result:** 3 of 4 Olympic Games qualify as Ford records

### Temporal Metrics Added to Data

Each unattached/international swim includes these analysis fields:

```python
ford_year: boolean
    # True if swimmer had ≥1 Ford swim in this year
    
ford_affiliation_status: string
    # "During-Ford-Year" | "Pre-Ford" | "Post-Ford" | "Between-Ford-Periods"
    
days_since_ford_swim: integer
    # Days since nearest Ford swim before this swim
    
days_until_ford_swim: integer  
    # Days until nearest Ford swim after this swim
    
ford_swim_before_date: date
    # Date of nearest Ford swim before this swim
    
ford_swim_after_date: date
    # Date of nearest Ford swim after this swim
```

---

## Record Compilation Process

### Step 1: Data Consolidation

**Combine all qualifying swims:**

1. **Official Ford swims** → `data/raw/scy/` and `data/raw/lcm/`
2. **Probationary swims** → `data/raw/scy-unattached-probationary/`
3. **Ford-bounded swims** → `data/raw/scy-unattached-other/`
4. **International swims** → `data/raw/lcm-international/`

**Consolidation script:** `scripts/generate_combined_event_files.py`

**Output:** `data/processed/{course}/{course}-{gender}-{age}-{event}.csv`

**Statistics:**
- SCY: 335 event files, 57,148 swims
- LCM: 321 event files, 43,721 swims  
- SCM: 73 event files, 498 swims (all unattached/international)

### Step 2: Event File Organization

**File naming convention:**
```
{course}-{gender}-{age_group}-{event}.csv

Examples:
  scy-m-15-16-100-free.csv
  lcm-f-Open-200-back.csv
  scy-m-13-14-relay-400-free.csv
```

**Age group handling:**

For swimmers ≤18 years old, swims are **duplicated** into both age group file AND Open file:
```python
def determine_age_groups(age):
    age_groups = []
    
    if age <= 10:
        age_groups.append('10U')
    elif age <= 12:
        age_groups.append('11-12')
    elif age <= 14:
        age_groups.append('13-14')
    elif age <= 16:
        age_groups.append('15-16')
    elif age <= 18:
        age_groups.append('17-18')
    
    # All swimmers also go in Open
    age_groups.append('Open')
    
    return age_groups
```

**Example:** A 15-year-old's 100 free swim appears in:
- `scy-m-15-16-100-free.csv` (age group file)
- `scy-m-Open-100-free.csv` (Open file)

For swimmers >18, swims only appear in Open file.

### Step 3: Record Identification

**Process:**

1. **Load event file:** All swims for specific course/gender/age/event
2. **Sort by time:** Fastest swims first
3. **Identify records:**
   - **Team record:** Single fastest time (all-time)
   - **Top 10:** Ten fastest swimmers (best time per swimmer)
   - **Top 10 swims:** Ten fastest swims (allows multiple per swimmer)

**Script:** `scripts/generate_records_and_top10.py`

**Example SQL-like query:**
```sql
-- Team record (fastest time ever)
SELECT TOP 1 
    Name, SwimTime, Age, Meet, SwimDate, UnattachedType
FROM event_file
ORDER BY SwimTime ASC

-- Top 10 swimmers (best time per swimmer)
SELECT TOP 10
    Name, MIN(SwimTime) as BestTime, Age, Meet, SwimDate, UnattachedType
FROM event_file
GROUP BY PersonKey
ORDER BY BestTime ASC

-- Top 10 swims (multiple per swimmer allowed)
SELECT TOP 10
    Name, SwimTime, Age, Meet, SwimDate, UnattachedType
FROM event_file  
ORDER BY SwimTime ASC
```

### Step 4: Indicator Assignment

**Swim indicators added based on data source:**

```python
def assign_indicator(swim):
    if swim.team == "Tucson Ford Dealers Aquatics":
        return ""  # No indicator (official Ford)
    elif swim.unattached_type == "Probationary":
        return "(‡)"  # Probationary period
    elif swim.unattached_type == "Other":
        return "(†)"  # Ford-bounded unattached
    elif swim.data_source == "International":
        return "(◊)"  # International competition
```

**Indicator meanings:**
- **(no indicator)** = Official Ford team swim
- **(‡)** = Probationary period (before joining Ford)
- **(†)** = Ford-bounded unattached (college, trials, etc.)
- **(◊)** = International (Olympics, Worlds, World Juniors)

### Step 5: Output Formatting

**Markdown files generated:**

**Individual Records:**
```markdown
# Boys 15-16 100 Freestyle - SCY

**Team Record:** 45.23 - John Smith (‡) - 2024 Winter Sectionals - 01/15/2024

## Top 10 All-Time

1. 45.23 - John Smith (‡) - Age 15 - 2024 Winter Sectionals - 01/15/2024
2. 45.67 - Mike Jones - Age 16 - 2023 State Championships - 03/10/2023
3. 46.01 - Tom Brown (†) - Age 15 - 2022 NCAA Regionals - 02/20/2022
...
```

**Relay Records:**
```markdown
# Boys 15-16 400 Free Relay - SCY

**Team Record:** 3:12.45 - 2024 State Championships - 03/15/2024
  Relay Team: Smith, Jones, Brown, Davis

## Top 10 All-Time

1. 3:12.45 - 2024 State Championships - 03/15/2024
2. 3:13.22 - 2023 Winter Invite - 01/20/2023
...
```

---

## Quality Assurance

### Data Validation

**Automated checks:**

1. **Duplicate detection:** `UsasSwimTimeKey` prevents duplicate swims
2. **Time validation:** All times verified as valid swim format (MM:SS.HH or SS.HH)
3. **Age consistency:** Age matches year of birth (when available)
4. **Date validation:** All dates in valid YYYY-MM-DD format
5. **Team affiliation:** Ford swimmers verified against roster

**Manual verification:**

1. **Record spot checks:** Random sample verification against published records
2. **Elite swimmer validation:** International results cross-checked with Olympics.com
3. **Meet date verification:** Major meets verified against USA Swimming calendar
4. **Unattached classification:** Sample reviews of probationary/Ford-bounded classifications

### Known Limitations

1. **Pre-1998 records:** USA Swimming database incomplete before 1998
2. **Missing meets:** Some small invitationals may not be in database
3. **Relay team members:** Individual names not always captured for relays
4. **International coverage:** Only includes swimmers with World Aquatics profiles
5. **SCM data:** Very limited (most competitions are SCY or LCM in USA)

---

## Technical Implementation

### Software Stack

**Language:** Python 3.x

**Key Libraries:**
- `pandas` - Data manipulation and analysis
- `requests` - API communication
- `json` - Data format handling
- `pathlib` - File system operations
- `datetime` - Date/time manipulation

### Scripts

**Data Collection:**
- `usa_swimming_api.py` - USA Swimming API client
- `world_aquatics_api.py` - World Aquatics API client
- `collect_team_results.py` - Bulk Ford swim collection
- `download_ford_international_results.py` - International results downloader

**Swim Classification:**
- `classify_unattached_swims.py` - Probationary vs Ford-bounded classification
- `organize_unattached_swims.py` - Integrate unattached into event files
- `organize_international_swims.py` - Integrate international into event files

**Record Generation:**
- `generate_combined_event_files.py` - Consolidate all data sources
- `generate_records_and_top10.py` - Extract records and rankings
- `consolidate_year_files.py` - Combine multi-year data

**Data Management:**
- `update_unattached_types.py` - Add indicator columns
- `remove_unattached_from_raw.py` - Clean raw data files
- `consolidate_scm_files.py` - Consolidate SCM results

### Data Architecture

```
data/
├── raw/                                    # Original data from APIs
│   ├── scy/                               # Official Ford SCY swims
│   ├── lcm/                               # Official Ford LCM swims
│   ├── scm/                               # Official Ford SCM swims (empty)
│   ├── scy-unattached-probationary/       # Probationary period swims
│   ├── lcm-unattached-probationary/
│   ├── scm-unattached-probationary/
│   ├── scy-unattached-other/              # Ford-bounded unattached
│   ├── lcm-unattached-other/
│   ├── scm-unattached-other/
│   └── international/                     # International competition results
│
├── processed/                             # Combined event files
│   ├── scy/                               # All SCY swims (4 sources combined)
│   ├── lcm/                               # All LCM swims (4 sources combined)
│   ├── scm/                               # All SCM swims (unattached only)
│   └── unattached/                        # Individual swimmer classifications
│       ├── probationary/                  # Probationary swimmers CSVs
│       └── ford-unattached/               # Ford-bounded swimmers CSVs
│
└── records/                               # Final published records
    ├── new_records_scy_boys.md
    ├── new_records_scy_girls.md
    ├── top10_scy_boys.md
    └── ... (12 total files)
```

### Reproducibility

**Complete workflow to regenerate records:**

```bash
# 1. Clone repository
git clone https://github.com/aaryno/ford-data-analysis.git
cd ford-data-analysis

# 2. Install dependencies  
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 3. Collect USA Swimming data (27 years)
for year in {1998..2025}; do
    python scripts/collect_year.py $year scy
    python scripts/collect_year.py $year lcm
done

# 4. Download swimmer profiles (for unattached classification)
python scripts/download_all_swimmer_profiles.py

# 5. Classify unattached swims
python scripts/classify_unattached_swims.py

# 6. Download international results
python scripts/download_ford_international_results.py

# 7. Organize all data sources
python scripts/organize_unattached_to_events.py both
python scripts/organize_international_swims.py

# 8. Consolidate into event files
python scripts/generate_combined_event_files.py

# 9. Generate records
python scripts/generate_records_and_top10.py

# Records output: data/records/*.md
```

**Total processing time:** ~4-6 hours (depending on API response times)

---

## Version History

**Version 1.0 (October 2025)**
- Initial implementation
- USA Swimming data: 1998-2025 (27 years)
- International data: 5 elite swimmers, 900 swims
- Strict Temporal Boundaries affiliation rule
- 282,086 total swims analyzed
- 594 international swims qualified (66%)

---

## Contact

For questions about methodology or data accuracy:

**Tucson Ford Dealers Aquatics**  
Website: https://www.gomotionapp.com/team/astfda/page/home

**Data Analysis Team**  
Repository: https://github.com/aaryno/ford-data-analysis (private)  
Public Records: https://github.com/aaryno/tucson-ford-dealers-aquatics-records

---

**Document Version:** 1.0  
**Last Updated:** October 2025  
**Status:** Published Draft
