# Friends TV Show — Dialogue Dataset

A cleaned, analysis-ready dataset of dialogue lines from all 10 seasons of *Friends* (1994–2004), built from HTML episode transcripts via web scraping.

---

## Files

| File | Rows | Description |
|---|---|---|
| `friends_clean.csv` | 54,008 | Main dataset — all canonical episodes, cleaned |
| `friends_special_episodes.csv` | 177 | Unmatched rows (Conan O'Brien special + misc.) |

---

## Dataset at a Glance

- **Seasons:** 1–10
- **Episodes:** 214
- **Writers:** 63 unique credited writers
- **Locations:** 16 canonical locations (normalised from 100+ raw variants)
- **Main cast lines:** Rachel 8,558 · Ross 8,403 · Monica 7,765 · Chandler 7,772 · Joey 7,599 · Phoebe 6,866

---

## Columns

| Column | Type | Description |
|---|---|---|
| `Season` | int | Season number (1–10) |
| `Episode` | int | Episode number within the season |
| `EpisodeLabel` | str | Formatted label e.g. `S01E01` |
| `WrittenBy` | str | Episode writer(s), normalised |
| `Scene` | str | Raw scene header from transcript |
| `Location` | str | Canonical location (see Location Mapping below) |
| `Character` | str | Character name, lowercased |
| `IsMainCharacter` | bool | True for the 6 main cast members |
| `IsChorusLine` | bool | True for lines attributed to `all` (group chants etc.) |
| `Dialogue` | str | Cleaned dialogue text |
| `WordCount` | int | Word count of the dialogue line |

---

## Source

Transcripts were scraped from episode HTML files. Each episode's full transcript was parsed to extract scene headers, character attributions, and dialogue line by line.

---

## Cleaning Steps

The raw data had several issues that were fixed in `cleaning.py`:

**Dialogue text**
- 15,423 lines contained embedded `\r\n` carriage returns (line wraps from the HTML source). All collapsed to single-line clean text.
- Windows-1252 smart quote artifacts (`\x92`) replaced with standard apostrophes throughout.

**Character names**
- Lowercased and stripped of whitespace across all rows.
- `mnca` → `monica` (recurring typo in source transcripts)
- Other fixes: `racheal`, `phoebs`, `chandlerv`, `dr. geller`, `joey tribbiani`, `mr. geller`
- Lines attributed to `all` (group responses) flagged via `IsChorusLine` rather than dropped.

**Location**
- Raw `Scene` field contained 100+ inconsistent location strings (e.g. `Monica and Rachel's`, `Monica and Chandlers`, `Monica and Chandler's apartment`, `Monica` — all the same place).
- Extracted via regex from the `[Scene: ...]` header pattern.
- Normalised into 16 canonical locations using a manually curated mapping.
- Locations with fewer than 200 lines consolidated into `Other`.

**WrittenBy**
- Leading/trailing whitespace and trailing periods stripped.
- Mixed separators (`and`, `,`) standardised to ` & `.
- Misspellings fixed: `Kaufmann` → `Kauffman`, `Astroff` → `Astrof`, `Jeff Greenstein` → `Jeffrey Greenstein`.
- Reduced from 85 raw variants to 63 clean unique writers.

**Added columns**
- `WordCount` — computed from cleaned `Dialogue`
- `IsMainCharacter` — True for rachel, ross, chandler, monica, joey, phoebe
- `IsChorusLine` — True where Character == 'all'
- `EpisodeLabel` — formatted `SxxExx` string for easy filtering

**Excluded rows**
- 177 rows with null Season/Episode (a Conan O'Brien behind-the-scenes special and a handful of unmatched lines) saved separately in `friends_special_episodes.csv`.
- 6 rows with null/empty Dialogue dropped.

---

## Known Limitations

- **Season 2 is incomplete** — only 12 of ~24 episodes were successfully scraped. Treat Season 2 trends as partial data.
- **`Other` location bucket** is large (~12K rows) and includes many one-off locations (Barbados, various hotels, the rest stop etc.). Fine for aggregate analysis; not suitable for granular location work.
- **Scene field has 388 nulls** — rows where no scene header appeared in the transcript before that line. Location is set to `Unknown` for these.
- Dialogue is as written in fan-maintained transcripts and may contain minor transcription errors from the original source.

---

## Location Mapping (Canonical)

| Canonical Name | Covers |
|---|---|
| Monica's Apartment | Monica & Rachel's, Monica & Chandler's, all bedroom/kitchen variants |
| Central Perk | All Central Perk variants incl. `Ross is in Central Perk` |
| Chandler & Joey's Apartment | Chandler and Joey's, At Chandler and Joey's, etc. |
| Joey's Apartment | Joey & Rachel's (later seasons) |
| Ross's Apartment | Ross's, Ross', Ross & Rachel's |
| Phoebe's Apartment | Phoebe's, Phoebe & Rachel's |
| Hospital | Delivery room, labor room, recovery room, emergency room, nursery |
| The Hallway | Hallway, corridor, between the apartments |
| Restaurant | A restaurant, The restaurant, Moondance Diner |
| Exterior / Street | A street, Outside Central Perk, The park |
| Rachel's Office / Room | Bloomingdale's, Ralph Lauren, Rachel's office, Rachel's room |
| Chandler's Office | Chandler's office, office in Tulsa, conference room in Tulsa |
| Away / Travel | Barbados, beach house, casino, cabin, rest stop |
| Film / TV Set | Silvercup Studios, movie set, the set |
| Carol & Susan's | Carol and Susan's |
| Airport | Airport, boarding scenes |

---

## Usage

```python
import pandas as pd

df = pd.read_csv('friends_clean.csv')

# Main cast only
main = df[df['IsMainCharacter']]

# Lines per character per season
main.groupby(['Season', 'Character'])['Dialogue'].count()

# Total words spoken
main.groupby('Character')['WordCount'].sum().sort_values(ascending=False)

# Top locations
df['Location'].value_counts()
```

---

## Project Context

This dataset was built as part of a data analytics portfolio project. The full project includes a 4-page Power BI dashboard covering character analysis, season/episode breakdowns, location analysis, and word count depth.

**Stack:** Python (pandas, re) · Power BI · Web scraping (BeautifulSoup / requests)

---

## Acknowledgements

Transcript data sourced from fan-maintained HTML episode transcripts. All content rights belong to Warner Bros. / Bright/Kauffman/Crane Productions. This dataset is for personal educational and portfolio use only.
