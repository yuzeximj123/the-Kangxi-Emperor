<p align="center">
  <img src="assets/cover.jpg" alt="Portrait of the Kangxi Emperor in court dress" height="520">
</p>

<h1 align="center">Kangxi Governance Corpus</h1>

<p align="center">
  <a href="README.zh-CN.md">简体中文</a> · <b>English</b>
</p>

A digitized, searchable, cross-linked corpus of the primary sources on how the **Kangxi Emperor (r. 1661–1722)** governed — his daily court record and his personally written vermilion rescripts (朱批) — turned into an event database you can query.

> Cover: *Portrait of the Kangxi Emperor in Court Dress* (清圣祖康熙皇帝朝服像), a Qing court painting.

## Highlights: what the data shows

A quick tour of what you can find here (all figures computed from `events.db`; see [Known limitations](#known-limitations)).

**What an emperor's work looks like, by the numbers.** Of 54,401 events, the largest categories are the emperor's movements (7,772), court sessions (6,773), edicts and decisions (6,095) and appointments and dismissals (5,193). Legal cases (2,946) outnumber military affairs (1,302) more than two to one: this was mostly a government of paperwork, not of campaigns.

**Who appears most.** Apart from the emperor himself (30,043 event appearances) and the Grand Empress Dowager (1,209), the most frequent names are the Grand Secretaries 明珠 Mingju (2,016), 伊桑阿 Isangga (1,884), 库勒纳 Kulena (1,336), 张玉书 Zhang Yushu (1,209) and 马齐 Maci (1,077).

**When the state was busiest.**

| Theme | Peak reign years | Events |
|---|---|---|
| Military (军务) | year 35 (1696) and 36 (1697): the Dzungar campaign against 噶尔丹 Galdan | 241, 118 |
| River works (河工水利) | year 39 (1700), then 27 (1688) | 82, 58 |
| Appointments (人事任免) | years 19–25, around the consolidation of the 1680s | 239–281 per year |

Galdan alone is mentioned in 440 event summaries.

**The emperor's own voice.** 6,193 events carry a vermilion rescript, and the rescript summaries preserve how he wrote to officials. The most common formula is a personal health report: *"朕体安"* ("I am well"). A few examples:

- Reign year 32, on a harvest report: *"Changshu reports the grain in Shaanxi is very good, and I am very pleased."*
- Reign year 45, hunting: *"Frost has not yet fallen and the stags are not calling. I have begun the hunt, but the mosquitoes are unbearable."*
- Reign year 45, on a prefect's appointment: *"The prefect was chosen by me. I do not know these two men. If I used them on your word, would that be right?"*
- Reign year 58, to an official who sent gifts: *"Do not send me presents again. If you need something, write to me."*

Rainfall and harvest are a recurring subject: 1,168 rescripted memorials mention rain.

**Long case threads.** `event_threads` follows one matter across the three corpora. The longest, thread 3 (361 events), traces the decision to campaign personally against Galdan. Others are the 噶礼 Geli vs. 张伯行 Zhang Boxing impeachment affair (threads 2, 17 and 22), the 赵申乔 Zhao Shenqiao father-and-son corruption dispute (thread 7), and the Fujian coastal unrest reports (thread 4). Try:

```sql
SELECT thread_id, member_count, corpora, label
FROM event_threads ORDER BY member_count DESC LIMIT 10;
```

**Where things happened.** Beyond the palace gates (乾清门, 4,285 events), the most frequent places are 江南 (945), 畅春园 (892), 直隶 (886), 福建 (842) and 浙江 (838): the Lower Yangtze and the coast dominate the provincial record.

## What is in this repository

Three primary-source collections, processed through OCR, cleaning and structured extraction:

| Corpus | Content | Volumes |
|---|---|---|
| 起居注 (Court Diaries) | Daily record of the emperor's audiences, edicts, appointments and movements | 8 |
| 汉文朱批奏折 (Chinese vermilion-rescripted memorials) | Memorials from officials with the emperor's handwritten replies, reign years 28–61 | 8 |
| 满文朱批奏折 (Manchu vermilion-rescripted memorials, Chinese translation) | Memorials originally in Manchu, with rescripts | 1 (large) |

The result is a SQLite database, `event/events.db`, built on top of the cleaned texts.

### Database contents (`event/events.db`)

| Table | Rows | Description |
|---|---:|---|
| `events` | 54,401 | One row per event: 46,316 from 起居注, 5,025 from Manchu memorials, 3,060 from Chinese memorials. Includes date, event role, summary, and rescript summary |
| `people` / `person_aliases` | 26,526 / 27,979 | Canonical person names, with aliases and homonym splits (`person_splits`, `person_identity_notes`) |
| `event_people` | 146,409 | Which people take part in each event, and in what role |
| `person_posts` | 111,432 | Offices held by each person over time, with normalized office names and categories |
| `places` / `event_places` | 10,771 / 72,178 | Canonical places and where events occurred |
| `event_links` | 625,455 | Candidate links between events, with a relation type and confidence |
| `event_threads` / `event_thread_members` | 213 / 1,290 | Chains of linked events across corpora, e.g. one case from first report to final outcome |

Event roles include 皇帝行止 (the emperor's movements), 议政听政 (court sessions), 谕旨决策 (edicts and decisions), 人事任免 (appointments and dismissals), 司法案件 (legal cases), 河工水利 (river works), 军务 (military) and others.

## Repository layout

```
assets/                  Cover image
raw/                     Source PDFs (not tracked; see "Sources")
processed/               OCR output and tables of contents per corpus
output/
  原稿/                   Raw OCR text
  简体中文版/              Cleaned, punctuated, Simplified Chinese text
  现代白话文/              Modern vernacular Chinese rendering
event/
  events.db              The event database (tracked with Git LFS)
  export/                events.csv and events.json exports
  staging/               Intermediate extraction JSON, per corpus
_archive/                Scratch work and old logs
```

## Getting started

The database is stored with [Git LFS](https://git-lfs.com/), so install it before cloning.

```bash
git lfs install
git clone https://github.com/yuzeximj123/the-Kangxi-Emperor.git
cd the-Kangxi-Emperor
```

Query it with any SQLite client:

```bash
sqlite3 event/events.db
```

```sql
-- Appointments and dismissals by reign year
SELECT reign_year, COUNT(*) FROM events
WHERE event_role = '人事任免' GROUP BY reign_year ORDER BY reign_year;

-- Which people appear most in a given period
SELECT p.canonical_name, COUNT(*) AS n
FROM event_people ep
JOIN events e USING (event_id)
JOIN people p USING (person_id)
WHERE e.reign_year BETWEEN 21 AND 30
GROUP BY 1 ORDER BY n DESC LIMIT 10;

-- Vermilion rescripts mentioning rainfall
SELECT reign_year, zhupi_summary FROM events
WHERE has_zhupi = 1 AND zhupi_summary LIKE '%雨%';

-- Walk one event thread in time order
SELECT e.source_corpus, e.reign_year, e.month, e.summary
FROM event_thread_members m JOIN events e USING (event_id)
WHERE m.thread_id = 7
ORDER BY e.reign_year, e.month, e.order_index;
```

On Windows consoles, set `PYTHONIOENCODING=utf-8` if you read it from Python, or Chinese output will be garbled.

## Known limitations

Please read before drawing conclusions from the data.

- **OCR and extraction errors.** Text comes from OCR of scanned volumes and automated structured extraction. Names, dates and offices can be wrong.
- **Homonyms.** People with the same name may be merged into one record; some have been split (`person_splits`), but not all. Examples seen in the data: early offices attributed to a person who held them decades later.
- **Year labels.** Some records carry an incorrect or missing `reign_year`. For example, the Manchu volume has an unusually large cluster at year 3.
- **Event links are model-generated.** `event_links` and `event_threads` are inferred, with confidence around 0.8 on average. Treat them as leads to verify, not facts.
- **Counts are indicative.** Keyword counts in summaries are approximate; one record can match several keywords.
- **Coverage differs by period.** The court diaries start effectively from reign year 9–10, and the Chinese memorials from year 28.

## Sources

The source texts are published compilations of the First Historical Archives of China and the Court Diaries of the Kangxi reign. The original PDFs are **not** included in this repository (`raw/` is git-ignored). The compilations are the work of their editors and publishers; the transcriptions here are provided for research.

## License

Code and derived data in this repository are released under the [GNU Affero General Public License v3.0](LICENSE). That license does not change the rights that belong to the original compilers and publishers of the source texts.
