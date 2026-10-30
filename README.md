<p align="center">
  <img src="assets/cover.jpg" alt="Portrait of the Kangxi Emperor in court dress" height="520">
</p>

<h1 align="center">Kangxi Governance Corpus</h1>

<p align="center">
  <a href="README.zh-CN.md">简体中文</a> · <b>English</b>
</p>

A digitized, searchable, cross-linked corpus of the primary sources on how the **Kangxi Emperor (r. 1661–1722)** governed — his daily court record and his personally written vermilion rescripts (朱批) — turned into an event database you can query.

> Cover: *Portrait of the Kangxi Emperor in Court Dress* (清圣祖康熙皇帝朝服像), a Qing court painting.

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
