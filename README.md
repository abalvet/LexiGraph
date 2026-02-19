# LexiGraph:  a Serious Game for Building Semantic Links

**A collaborative lexical game based on the TLFi dictionary (ATILF XML release)**

LexiGraph lets players connect distant words through meaningful intermediate steps. While playing, users collectively build a graph of semantic relations extracted from real dictionary data.

**Example challenge:**

```
LIGNIFIER → BOIS → FORÊT → OMBRE → FRAÎCHEUR → GLACE
(to lignify) → (wood) → (forest) → (shadow) → (freshness) → (ice)
```

Shorter chains = better score.  
Every validated relation enriches a lexical knowledge base.

---

## What This Project Demonstrates

LexiGraph is both a game and a research/teaching tool. It shows how structured lexical resources (XML dictionaries) can be turned into:

- An interactive web application
- A navigable semantic graph
- A crowdsourced linguistic dataset

The game uses the [newly released XML](https://www.ortolang.fr/market/lexicons/xml-tlfi/v1)  version of the [TLFi dictionary](https://www.cnrtl.fr/definition/) (Trésor de la Langue Française informatisé).

---

## Project Structure

```
LexiGraph/
├── standalone/                    # Offline demo (limited vocabulary and challenges)
│   └── index.html
├── server/                        # Flask application
│   ├── app.py
│   └── templates/
│       └── index.html             # Full interface (91,000+ words)
├── scripts/
│   ├── tlfi_to_json_v2.py         # XML → JSON converter (hierarchical)
│   ├── merge_tlfi_json.py         # Merges multiple JSON files
│   └── convert_and_merge_tlfi.sh  # Full conversion pipeline
├── data/                          # Generated data (not included)
│   └── tlfi_sample.json
├── requirements.txt
└── README.md
```

---

## Quick Start (Server Mode)


### Overview

1. Convert the TLFi XML files into JSON
2. Install Python dependencies
3. Launch a local web server
4. Play in your browser

**Total time:** ~10 minutes (excluding TLFi download)

---

### Step 1:  Get the Project

```bash
git clone https://github.com/YOUR_USERNAME/LexiGraph.git
cd LexiGraph
```

Or download and extract the ZIP archive.

---

### Step 2:  Set Up Python Environment

**Requirements:** Python 3.8+

Create and activate a virtual environment (eg. use "lexigraph" instead of "venv"):

```bash
# Linux / macOS
python3 -m venv venv
source venv/bin/activate

# Windows
python -m venv venv
venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

### Step 3:  Download the TLFi XML

The game requires the ATILF TLFi XML release:

**https://www.ortolang.fr/market/lexicons/xml-tlfi/v1**

You will get > 80 XML files.

Alternatively: you can use the JSON conversion of the current version. But **remember**: this JSON version matches the 2026-02 XML version of TLFi.

---

### Step 4:  Convert the TLFi to JSON

From the project root:

```bash
chmod +x scripts/convert_and_merge_tlfi.sh
./scripts/convert_and_merge_tlfi.sh /path/to/tlfi_xml/ data/
```

This script:
1. Converts each XML file to JSON (preserving hierarchical sense structure)
2. Merges all files into a single dataset
3. Produces `data/tlfi_complet.json`

Then rename for the server:

```bash
mv data/tlfi_complet.json data/tlfi_sample.json
```

**Verify the conversion:**

```bash
python -c "import json; d=json.load(open('data/tlfi_sample.json')); print(f'{len(d[\"entries\"])} entries loaded')"
```

---

### Step 5:  Start the Server

```bash
cd server
python app.py
```

You should see:

```
✓ Lexicon loaded: 91476 entries
✓ Database: /path/to/LexiGraph/data/LexiGraph.db
✓ Server started on http://localhost:5000
```

---

### Step 6:  Play

Open your browser:

```
http://localhost:5000
```

You can now:

- Generate random challenges
- Build new semantic links 
- Explore word neighbors (derivatives, related terms)
- Submit your links

All proposed relations are stored locally in a SQLite database.

---

### Stopping and Restarting

**Stop the server:** `Ctrl + C`

**Restart later:**

```bash
cd LexiGraph
source venv/bin/activate   # Linux/macOS
cd server
python app.py
```

---

## Standalone Mode (No Server)

For a quick demo without installation, open:

```
standalone/index.html
```

This version:

- Works offline
- Requires no installation
- Uses a limited built-in vocabulary (~25 words)
- Does not save player contributions

---

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/challenge` | GET | Generate a new word pair |
| `/api/word/<word>` | GET | Get word info (definition, derivatives, relations) |
| `/api/check-relation` | POST | Check if two words are already linked |
| `/api/submit-chain` | POST | Submit a completed chain |
| `/api/submit-relation` | POST | Submit a single relation |
| `/api/export/relations` | GET | Export all collected relations (JSON) |
| `/api/stats` | GET | Game statistics |
| `/api/leaderboard` | GET | Player rankings |

---

## What the Game Collects

The server stores in SQLite:

- **Proposed relations**:  with player ID, timestamp, and vote counts
- **Completed chains**:  full path, score, player
- **Aggregated votes**:  relations confirmed by multiple players gain trust

### Export for Analysis

```bash
curl http://localhost:5000/api/export/relations > relations.json
```

This allows linguists to:

- Review proposed relations
- Validate or reject them
- Integrate confirmed relations into lexical resources

---

## Why TLFi XML Matters

A dictionary is not just text.

In XML format, each TLFi entry contains structured linguistic information:

| Element | Example |
|---------|---------|
| Lemma | LIGNIFIER |
| Part of speech | verbe pronominal |
| Domain | BOT. (botany) |
| Definition | Se transformer en bois |
| Literary examples | With author, title, date |
| Derivatives | LIGNIFIÉ, LIGNIFICATION |
| Etymology | Latin *lignum* (wood) |

LexiGraph extracts these relations and turns them into a navigable semantic network. The game then lets users **discover relations the dictionary doesn't explicitly encode**:  synonyms, , hypernyms, associations, etc.

---

## Known Limitations

1. **Some challenges are unsolvable.** Lexical graphs have a "small world" structure: dense clusters connected by few bridges. If two words belong to distant clusters, no natural semantic chain exists. But maybe this a feature and not a problem? 

2. **Relations are untyped.**  In the current version, the game collects that A→B are related, but not *how* (synonym? hypernym? association?). Projects like [JeuxDeMots](http://www.jeuxdemots.org/) use 180+ relation types.

3. **Single-user prototype.** No authentication, no anti-cheating mechanisms. For production use, the stack would need hardening. Like, a lot.

---

## Educational Use

This project was developed in the context of the **LIIAN Master's program** (Linguistique Informatique pour l'Inclusion et l'Accessibilité Numérique) at the University of Lille.

It illustrates core skills taught in the program:

| Skill | Application in LexiGraph |
|-------|---------------------------|
| XML processing | Parsing TLFi structure |
| Python scripting | Conversion pipeline |
| Web APIs | Flask REST endpoints |
| Databases | SQLite storage |
| Data visualization | Graph rendering (Canvas) |
| Lexical semantics | Relation extraction |
| Crowdsourcing | Collaborative annotation |

---

## License

**Lexical data:** TLFi:  [ATILF](http://www.atilf.fr/) (CNRS – Université de Lorraine)  
**Code:** MIT License

---

## Credits

Developed by **Antonio BALVET** in the context of the LIIAN Master's program:  University of Lille and UMR STL 8163.

Inspired by:
- [JeuxDeMots](http://www.jeuxdemots.org/) (LIRMM, Montpellier)
- [Semantris](https://research.google.com/semantris/) (Google)

---

## Contributing

Contributions are welcome! Possible improvements:

- [ ] Filter challenges by feasibility (BFS path check)
- [ ] Add relation typing (synonym, hypernym, etc.)
- [ ] Implement word autocompletion
- [ ] Visualize subgraph between start and end words
- [ ] Add multilingual support
- [ ] WCAG accessibility compliance

Please open an issue to discuss major changes before submitting a PR.
