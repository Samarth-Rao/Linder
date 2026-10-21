# Linder

Measuring orthographic and phonetic similarity between English and 96 languages across 27 language families.

Each English word is paired with its translation, romanized if needed, and scored on how alike the two look and sound in Latin script.

## Live Site

**[View the interactive explorer →](https://samarth-rao.github.io/Linder/)**

## Key Findings

Family averages are the mean of each language's average composite score vs English, across the 92 languages with at least 1,000 translation pairs (Bengali, Marathi, Odia and Burmese have fewer and are excluded from the rankings).

| Family | Avg Score | Languages |
|--------|-----------|-----------|
| Romance | 0.48 | Catalan, Italian, French, Romanian, Portuguese, Spanish, Occitan, Galician, Sardinian |
| Germanic | 0.43 | Afrikaans, Dutch, Norwegian, Danish, Swedish, Norwegian Bokmal, German, Icelandic, Yiddish |
| Slavic | 0.35 | Bosnian, Croatian, Polish, Czech, Russian, Macedonian, Slovak, Bulgarian, Ukrainian, Slovenian, Belarusian, Serbo-Croatian |
| Indo-Iranian | 0.27 | Hindi, Persian, Urdu, Nepali, Punjabi, Gujarati, Sinhala, Tajik, Kurdish, Pashto |
| Semitic | 0.26 | Amharic, Maltese, Tigrinya, Hebrew, Arabic |
| Niger-Congo | 0.19 | Zulu, Yoruba, Shona, Kinyarwanda, Swahili, Igbo, Chichewa |

- **18% of German-English** translation pairs score ≥0.7 (high similarity)
- **Vietnamese** ranks highest overall (0.69) due to heavy Latin-script borrowing
- **Arabic** and **Hebrew** rank near the bottom (0.18, 0.20) — almost no orthographic overlap
- **Chinese** is bimodal: words are either phonetically borrowed (high) or completely different (low)

## How It Works

1. **Translation pairs** (English to each language) from several open sources, all normalized to `data/en-<code>.txt` (one tab- or space-separated pair per line):
   - [MUSE bilingual dictionaries](https://github.com/facebookresearch/MUSE) (Meta Research): 44 languages
   - [PanLex](https://panlex.org/) via the HuggingFace dataset [`cointegrated/panlex-meanings`](https://huggingface.co/datasets/cointegrated/panlex-meanings): 24 languages (`parse_panlex.py`)
   - [Wiktionary](https://kaikki.org/) English dump from kaikki.org, translation fields: 22 languages (`parse_wiktionary.py`)
   - [FreeDict](https://freedict.org/) dictd dictionaries: Welsh, Irish, Kurdish, Latin, Swahili and others (`parse_freedict.py`)
2. **Romanization** for non-Latin scripts: Pinyin (Chinese), IAST (Hindi/Bengali), Hepburn (Japanese), Unidecode (Cyrillic, Arabic, Greek, etc.)
3. **Composite score** — weighted blend of 7 metrics:
   - Normalized Levenshtein (30%) — edit distance
   - Jaro-Winkler (25%) — prefix-weighted similarity
   - Metaphone (15%) — English phonetic encoding
   - Soundex (10%) — coarse phonetic encoding
   - Bigram Jaccard (10%) — character pair overlap
   - Trigram Jaccard (5%) — character triple overlap
   - Length similarity (5%)

## Usage

```bash
# Compare two words
python main.py temperature temperatur

# Run a single language (10k pairs)
python batch.py --lang fr

# Run all 96 languages
python batch.py --lang all --limit 10000

# Generate charts
python visualize.py

# Export data for the website (needs results_*.csv from batch.py)
python export_web_data.py
python cross_compare.py
python export_lookup.py
python export_network.py   # reads web/data/summary.json + cross_matrix.json
```

### Supported Languages

```bash
python batch.py --list
```

Afrikaans, Albanian, Amharic, Arabic, Armenian, Azerbaijani, Basque, Belarusian, Bengali, Bosnian, Bulgarian, Burmese, Catalan, Chichewa, Chinese, Croatian, Czech, Danish, Dutch, Esperanto, Estonian, Filipino, Finnish, French, Galician, Georgian, German, Greek, Gujarati, Hausa, Hawaiian, Hebrew, Hindi, Hungarian, Icelandic, Igbo, Indonesian, Irish, Italian, Japanese, Kannada, Kazakh, Khmer, Kinyarwanda, Korean, Kurdish, Kyrgyz, Lao, Latin, Latvian, Lithuanian, Macedonian, Malay, Malayalam, Maltese, Maori, Marathi, Mongolian, Nepali, Norwegian, Norwegian Bokmal, Occitan, Odia, Pashto, Persian, Polish, Portuguese, Punjabi, Romanian, Russian, Samoan, Sardinian, Scottish Gaelic, Serbo-Croatian, Shona, Sinhala, Slovak, Slovenian, Somali, Spanish, Swahili, Swedish, Tajik, Tamil, Telugu, Thai, Tigrinya, Turkish, Ukrainian, Urdu, Uzbek, Vietnamese, Welsh, Yiddish, Yoruba, Zulu

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

mkdir -p data
```

The scoring scripts read translation pairs from `data/en-<code>.txt`. The `data/` folder is gitignored, so you need to build it once from these sources. Each source covers different languages; you only need the ones for the languages you want to run (`python batch.py --list` shows which dictionaries are present).

```bash
# 1. MUSE dictionaries (44 languages, direct download)
for lang in af ar bg bn bs ca cs da de el es et fa fi fr he hi hr hu id it ja ko lt lv mk ms nl no pl pt ro ru sk sl sq sv ta th tl tr uk vi zh; do
    curl -sL -o "data/en-${lang}.txt" "https://dl.fbaipublicfiles.com/arrival/dictionaries/en-${lang}.txt"
done

# 2. PanLex (24 languages, e.g. Amharic, Yoruba, Telugu, Odia).
#    Streams the dataset from HuggingFace; needs two extra packages.
pip install datasets pandas
python parse_panlex.py

# 3. Wiktionary (22 languages, e.g. Galician, Georgian, Marathi, Burmese).
#    The kaikki.org dump is several GB; the script expects it at /tmp/wiktionary-en.jsonl.gz.
curl -sL -o /tmp/wiktionary-en.jsonl.gz https://kaikki.org/dictionary/English/kaikki.org-dictionary-English.jsonl.gz
python parse_wiktionary.py

# 4. FreeDict (Welsh, Irish, Kurdish, Latin, Swahili, ...).
#    Download the dictd release of each eng-XXX dictionary from https://freedict.org/downloads/
#    and extract it so that <dir>/eng-XXX/eng-XXX.dict.dz and eng-XXX.index exist, e.g.:
mkdir -p /tmp/freedict
curl -sL https://download.freedict.org/dictionaries/eng-cym/0.2.3/freedict-eng-cym-0.2.3.dictd.tar.xz | tar -xJ -C /tmp/freedict
python parse_freedict.py /tmp/freedict
```

A few dictionaries were also supplemented by hand from [Apertium](https://www.apertium.org/) bilingual dictionaries (Basque, Maltese) and sinhala-para-dict (Sinhala); there is no script for those.

## Project Structure

```
├── main.py              # Single word-pair comparison (scoring metrics)
├── batch.py             # Batch processing (language registry, all languages)
├── cross_compare.py     # Pairwise language similarity matrix (English pivot)
├── visualize.py         # Generate matplotlib charts
├── parse_panlex.py      # Build data/en-XX.txt from PanLex (HuggingFace)
├── parse_wiktionary.py  # Build data/en-XX.txt from the kaikki.org Wiktionary dump
├── parse_freedict.py    # Build data/en-XX.txt from FreeDict dictd files
├── export_web_data.py   # Export summary + per-language JSON for web frontend
├── export_lookup.py     # Export word lookup dictionaries (web/data/lookup/)
├── export_network.py    # Export network graph data (web/data/network.json)
├── web/                 # Static website
│   ├── index.html
│   ├── style.css
│   ├── app.js
│   └── data/            # JSON data files
└── requirements.txt
```
