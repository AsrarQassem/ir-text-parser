# IR Text Parser

A Python-based **text parser** that serves as the foundational component of an Information Retrieval (IR) engine. The parser ingests TREC-formatted document collections, performs full linguistic preprocessing — tokenization, stopword elimination, and Porter stemming — and outputs structured term and document dictionaries ready for downstream indexing.

> Built as **Phase 1** of a multi-stage IR engine project for **CSCE 5200 — Information Retrieval and Web Search**.

---

## Overview

A complete Information Retrieval engine consists of three core components:

1. **Text Parser** — *this project*
2. **Indexer** — builds an inverted index from parsed terms
3. **Retrieval System** — processes user queries and ranks matching documents

This repository implements the **Text Parser**: the layer responsible for transforming raw, unstructured text into clean, normalized, ID-mapped tokens that the indexer and retrieval system can efficiently consume.

The parser is designed for the **TREC document format**, where each physical file contains *multiple* documents wrapped in `<DOC>` tags. The parser correctly splits these into individual documents, extracts each document's identifier from `<DOCNO>` tags, and produces two mapping dictionaries: one for terms and one for documents.

---

## Key Features

- **Multi-document file parsing** — uses regular expressions to split TREC files into individual documents based on `<DOC>` and `<DOCNO>` tags.
- **Tag-aware tokenization** — strips out `<DOCNO>` and `<PROFILE>` metadata before tokenization to prevent tag pollution in the term dictionary.
- **Strict tokenization rules** — splits on all non-alphanumeric characters, removes pure numeric tokens, discards any token containing digits, and lowercases all output.
- **Custom stopword removal** — filters tokens against a curated list of 523 common English stopwords loaded from `stopwordlist.txt`.
- **Porter stemming** — reduces inflected words to their root form using NLTK's `PorterStemmer` (e.g., *running, runs, ran* → *run*), increasing recall for downstream retrieval.
- **Dual dictionary output:**
  - **Term Dictionary** — maps each unique stemmed token to a unique numeric ID.
  - **Document Dictionary** — maps each TREC document name (e.g., `FT911-1`) to a unique numeric ID.
- **Alphabetically sorted output** — terms are sorted A–Z in the final output for easy inspection and verification.

---

## Tech Stack

| Component        | Technology                          |
|------------------|-------------------------------------|
| Language         | Python 3.10                         |
| NLP Library      | NLTK (`PorterStemmer`)              |
| Pattern Matching | `re` (regular expressions)          |
| Data Structures  | `collections.defaultdict`           |
| Environment      | Jupyter Notebook / CLI              |

---

## Project Structure

```
ir-text-parser/
│
├── Project_IR_Part1.ipynb     # Main Jupyter notebook with full pipeline
├── stopwordlist.txt           # Custom stopword list (523 words)
├── parser_output.txt          # Generated output (term + document dictionaries)
├── ft911/                     # Input TREC document collection
│   ├── ft911_1
│   ├── ft911_2
│   └── ... (ft911_3 through ft911_15)
└── README.md
```

---

## Pipeline

The parser executes the following preprocessing pipeline for each TREC file:

1. **File Ingestion** — reads each TREC file from the `ft911/` directory into memory.
2. **Document Segmentation** — extracts individual documents using `<DOC>...</DOC>` tag pairs.
3. **Document ID Extraction** — pulls each document's unique identifier from the `<DOCNO>` tag.
4. **Metadata Cleaning** — strips `<DOCNO>` and `<PROFILE>` content from the body so tag values don't end up in the term dictionary.
5. **Tokenization** — splits text on non-alphanumeric boundaries, lowercases all tokens, and discards any token containing non-alphabetic characters.
6. **Stopword Elimination** — filters tokens against the 523-word stopword set.
7. **Stemming** — applies the Porter algorithm to each surviving token.
8. **Dictionary Construction** — assigns unique sequential numeric IDs to new terms and documents.
9. **Sorting & Output** — alphabetizes the term dictionary and writes both dictionaries to `parser_output.txt`.

---

## Results

When run against the full input collection (15 files), the parser produced:

| Metric                             | Value      |
|------------------------------------|------------|
| Files processed                    | 15         |
| Documents parsed                   | **5,368**  |
| Unique stemmed terms extracted     | **32,896** |
| Total output entries               | **38,265** |

---

## Sample Output

The generated `parser_output.txt` contains the term dictionary first (alphabetically sorted), followed by a blank line separator, then the document dictionary:

```
aa              6520
aaa             13295
aachen          13912
aaron           17898
abandon         395
...

FT911-1         1
FT911-2         2
FT911-3         3
...
FT911-5368      5368
```

Each line follows the format:  `token<TAB><TAB>token_id`


---

## Design Decisions

- **Why Porter Stemming?** It's the de-facto standard in academic IR work and offers a strong balance between aggressiveness and accuracy for English text.
- **Why a custom stopword list?** Provides full control over which terms are filtered and removes the dependency on NLTK's stopword data download.
- **Why `defaultdict`?** Allows O(1) ID assignment without explicit key-existence checks, keeping the dictionary-construction logic concise.
- **Why strip `<DOCNO>` and `<PROFILE>` before tokenizing?** Prevents document IDs and metadata profile codes from polluting the term dictionary.
- **Why alphabetical sorting in the output?** Improves human readability and makes the output easier to inspect, debug, and verify.

---

## Roadmap

This parser is **Phase 1** of a larger IR engine. Future phases will build on this foundation:

- **Phase 2 — Indexer:** Construct an inverted index from the parsed tokens to enable fast term-to-document lookup.
- **Phase 3 — Retrieval System:** Implement query processing and ranked retrieval (e.g., TF-IDF, BM25) over the indexed corpus.

---

## License

This project is released for academic and educational purposes.
