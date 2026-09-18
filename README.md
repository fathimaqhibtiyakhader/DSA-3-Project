# DNA Marker Screening & Pattern-Matching System

## Overview

This project is a **Java-based DNA marker screening prototype** that detects predefined DNA sequence markers in a sample DNA sequence.

The system is designed around the **HBB (beta-globin) gene** and demonstrates how multiple DNA markers can be searched efficiently using the **Aho-Corasick string-matching algorithm**.

A **KMP (Knuth-Morris-Pratt)** implementation is also included as a baseline so that the two approaches can be compared for:

* Correctness
* Number of detected markers
* Search performance
* Scaling with increasing marker-panel size

> **Important:** This is an educational/prototype system using synthetic DNA data and teaching markers. It is **not a clinical diagnostic system** and its results must not be used for medical diagnosis.

---

# Project Objectives

The project demonstrates how a DNA screening pipeline can:

1. Represent DNA markers as nucleotide sequences.
2. Store metadata associated with each marker.
3. Scan a DNA sample for multiple markers simultaneously.
4. Identify the exact position of detected markers.
5. Associate detected sequences with diseases or genetic conditions.
6. Compare Aho-Corasick with KMP.
7. Generate reproducible synthetic DNA samples.
8. Demonstrate how the system could later be connected to real FASTA files.

---

# Technologies Used

* **Java**
* **Aho-Corasick Algorithm**
* **Knuth-Morris-Pratt (KMP) Algorithm**
* **FASTA sequence format**
* Object-oriented programming
* Java Collections Framework

No external Java libraries are required.

---

# Project Structure

```text
DNA/
│
├── AhoCorasick.java
├── BenchmarkDemo.java
├── FastaSampleGenerator.java
├── KMPMatcher.java
├── Marker.java
├── MarkerPanel.java
├── Match.java
├── ScreeningDemo.java
│
└── .vscode/
```

---

# File Descriptions

## 1. `Marker.java`

Represents a single DNA marker.

Each marker contains:

```text
id
pattern
gene
disease
mutationType
severity
```

For example:

```text
Marker ID: HBB_HbS
Gene: HBB
Disease: Sickle Cell Disease
Mutation Type: Point mutation
Severity: high
```

The class allows the sequence itself to be stored together with the metadata needed to generate a structured screening report.

---

# 2. `MarkerPanel.java`

Contains the project's predefined DNA marker panel.

The current panel contains three markers:

| Marker      | Gene | Description                                |
| ----------- | ---- | ------------------------------------------ |
| `HBB_HbS`   | HBB  | Sickle-cell-related marker                 |
| `HBB_BETA0` | HBB  | Synthetic beta-thalassemia teaching marker |
| `HBB_REF`   | HBB  | Wild-type reference/control marker         |

The panel acts as the **single source of truth** for the marker sequences and their metadata.

---

# 3. `Match.java`

Represents a marker detected in a DNA sequence.

Each match stores:

```text
markerId
position
matchedSubstring
```

Example:

```text
Match{marker=HBB_HbS, pos=80, pattern='CTGACTCCTGTGGAGAAGTCT'}
```

The position represents the **0-based starting position** of the match in the DNA sequence.

---

# 4. `AhoCorasick.java`

Implements the **Aho-Corasick multiple-pattern string matching algorithm**.

Instead of scanning the DNA sample separately for every marker, the algorithm constructs a trie containing all marker patterns.

It then creates:

* Trie transitions
* Failure links
* Output links/information

After construction, the DNA sequence can be scanned in essentially one pass.

### Basic workflow

```text
DNA markers
     │
     ▼
Build Trie
     │
     ▼
Build Failure Links
     │
     ▼
Aho-Corasick Automaton
     │
     ▼
Scan DNA sample
     │
     ▼
Detected markers
```

### Complexity

For a DNA sample of length `n` and total marker-pattern length `P`:

```text
Building automaton: O(P)
Searching:          O(n + number of matches)
```

This makes Aho-Corasick particularly useful when the same DNA sample needs to be screened against many markers.

---

# 5. `KMPMatcher.java`

Implements the **Knuth-Morris-Pratt algorithm**.

KMP is an efficient algorithm for searching for **one pattern at a time**.

The implementation:

1. Builds the LPS (Longest Proper Prefix which is also Suffix) table.
2. Scans the DNA sequence.
3. Detects occurrences of the marker.
4. Continues searching even when matches overlap.

For multiple markers, the sample is searched once for every marker.

Therefore, with `k` markers:

```text
Approximately O(k × n)
```

for patterns of comparable length, with the individual KMP search itself being linear.

KMP is therefore used as the **baseline comparison algorithm** in this project.

---

# 6. `FastaSampleGenerator.java`

Generates synthetic DNA sequences for testing.

The generator creates four samples:

```text
HBB_reference
sample_sickle_cell
sample_beta_thalassemia
sample_combined
```

The sequences are generated from a reproducible random DNA backbone.

The four samples represent:

### Reference

Contains the synthetic wild-type/reference sequences.

### Sickle-cell sample

Contains the synthetic sickle-cell variant at Hotspot 1.

### Beta-thalassemia sample

Contains the synthetic beta-thalassemia teaching variant at Hotspot 2.

### Combined sample

Contains both synthetic variants.

The generator can also write the sequences as FASTA files.

---

# 7. `ScreeningDemo.java`

Runs the complete screening demonstration.

It:

1. Loads the marker panel.
2. Builds the Aho-Corasick automaton.
3. Generates the four synthetic samples.
4. Searches every sample using Aho-Corasick.
5. Searches the same samples using KMP.
6. Compares the results.
7. Prints a structured report.

Example output format:

```text
=== sample_sickle_cell ===
Engines agree: true

[HBB_HbS] Sickle Cell Disease |
gene=HBB |
Point mutation (missense), beta-globin codon 6 Glu>Val |
severity=high |
pos=80
```

The important part of the demonstration is:

```text
Engines agree: true
```

This verifies that both implementations identified the same marker matches.

---

# 8. `BenchmarkDemo.java`

Tests how the two algorithms behave as the number of DNA markers increases.

The benchmark uses:

```text
5 markers
10 markers
20 markers
40 markers
80 markers
```

A synthetic DNA sequence of:

```text
50,000 nucleotides
```

is used for the benchmark.

The program reports execution time for:

```text
Aho-Corasick
KMP
```

Example table format:

```text
PanelSize   AhoCorasick(ms)   KMP(ms)
5           ...
10          ...
20          ...
40          ...
80          ...
```

The exact timings depend on the computer and Java runtime, so benchmark numbers should not be treated as universal performance measurements.

---

# How the System Works

The complete pipeline can be represented as:

```text
              DNA Marker Panel
                     │
                     ▼
             Marker Metadata
                     │
                     ▼
          ┌─────────────────────┐
          │  Aho-Corasick Trie  │
          └─────────────────────┘
                     │
              Failure Links
                     │
                     ▼
              Search Automaton
                     │
                     ▼
              DNA Sample
                     │
                     ▼
             Sequence Scanning
                     │
                     ▼
              Marker Matches
                     │
                     ▼
              Match Metadata
                     │
                     ▼
            Screening Report
```

---

# DNA Representation

DNA is represented as a string containing four nucleotide characters:

```text
A = Adenine
C = Cytosine
G = Guanine
T = Thymine
```

Example:

```text
ACACCATGGTGCATCTGACTCCTGTGGAGAAGTCT
```

The algorithms treat this as a string-search problem.

For example, if the marker is:

```text
CTGACTCCTGTGGAGAAGTCT
```

the program searches the DNA sample for that exact sequence.

---

# Marker Detection

Suppose the DNA sample contains:

```text
...ACACCATGGTGCATCTGACTCCTGTGGAGAAGTCT...
```

and the marker panel contains:

```text
CTGACTCCTGTGGAGAAGTCT
```

The matcher detects the marker and returns:

```text
Marker ID
Position
Matched DNA sequence
```

The `MarkerPanel` can then be used to retrieve additional information:

```text
Gene
Disease
Mutation type
Severity
```

This separates **sequence detection** from **marker interpretation**.

---

# Why Aho-Corasick?

A DNA screening system may eventually contain hundreds, thousands, or more marker patterns.

Searching independently using KMP would require repeatedly scanning the same DNA sequence:

```text
DNA
 │
 ├── Marker 1 → scan
 ├── Marker 2 → scan
 ├── Marker 3 → scan
 ├── Marker 4 → scan
 └── ...
```

Aho-Corasick instead combines the patterns:

```text
Marker 1 ─┐
Marker 2 ─┤
Marker 3 ─┼──► Trie + Failure Links ──► Single DNA Scan
Marker 4 ─┤
Marker 5 ─┘
```

This is the main algorithmic motivation for the project.

---

# Why KMP Is Included

KMP provides a useful baseline.

KMP is already efficient for a **single pattern**, with linear-time searching.

However, a multi-marker screening system needs to search for many patterns.

Therefore:

```text
KMP:
DNA → Marker 1
DNA → Marker 2
DNA → Marker 3
DNA → Marker 4
...
```

while:

```text
Aho-Corasick:
DNA → Marker 1 + Marker 2 + Marker 3 + Marker 4 + ...
```

The benchmark demonstrates this difference experimentally.

---

# Running the Project

## Requirements

Install:

* Java JDK 8 or later
* Terminal / command prompt
* Any Java-compatible IDE (optional)

Check Java:

```bash
java -version
```

Check the compiler:

```bash
javac -version
```

---

# Compile

Open a terminal inside the `DNA` directory.

Run:

```bash
javac *.java
```

If compilation succeeds, Java `.class` files will be generated.

---

# Run the Screening Demo

Run:

```bash
java ScreeningDemo
```

This performs the marker screening demonstration on the generated synthetic samples.

---

# Run the Benchmark

Run:

```bash
java BenchmarkDemo
```

This compares Aho-Corasick and KMP as the marker-panel size increases.

---

# Generate FASTA Files

The `FastaSampleGenerator` contains a method for writing the generated samples to FASTA files:

```java
FastaSampleGenerator.writeAll("data");
```

To use it, this call can be enabled from the appropriate Java program.

The generated directory will contain files such as:

```text
data/
├── HBB_reference.fasta
├── sample_sickle_cell.fasta
├── sample_beta_thalassemia.fasta
└── sample_combined.fasta
```

---

# FASTA Format

A generated FASTA file follows the standard structure:

```text
>sample_sickle_cell
ACGTACGTACGTACGT...
```

The first line beginning with `>` is the sequence identifier.

The following lines contain the nucleotide sequence.

`readFasta()` removes the header and reconstructs the DNA sequence into a single string for processing.

---

# Testing Strategy

The project uses two types of testing.

## Functional Testing

`ScreeningDemo.java` checks whether:

```text
Aho-Corasick results == KMP results
```

The results are converted into sets of `Match` objects and compared.

This helps verify that both implementations detect the same markers.

## Performance Testing

`BenchmarkDemo.java` measures the search time while increasing the number of markers.

The benchmark therefore evaluates how the algorithms behave as the marker panel grows.

---

# Synthetic Data

The current demonstration intentionally uses **synthetic DNA sequences**.

The synthetic generator uses deterministic random seeds:

```text
SEED = 2026
```

for sample generation.

This makes the experiment reproducible.

Running the program again produces the same generated sequences.

The benchmark also uses fixed seeds for reproducibility.

---

# Current Limitations

This project is a prototype and has several limitations.

### 1. Exact matching only

The current algorithms search for exact nucleotide strings.

They do not perform:

* Approximate matching
* Sequence alignment
* SNP tolerance
* Indel detection
* Sequencing-error correction

### 2. Synthetic teaching data

The current sample generator creates synthetic sequences.

Therefore, the results do not represent real patient sequencing results.

### 3. Limited marker panel

Only a small HBB-focused marker panel is currently implemented.

A production system would require a substantially larger, curated and validated marker database.

### 4. No clinical interpretation

A detected sequence match does not automatically establish:

* disease diagnosis
* disease severity
* carrier status
* clinical prognosis

Clinical interpretation requires validated laboratory methods, appropriate variant interpretation, quality controls, and qualified medical professionals.

### 5. No sequencing-quality information

Real sequencing data can contain:

* sequencing errors
* low-quality bases
* ambiguous bases
* coverage differences

The current implementation assumes a clean DNA string.

### 6. No biological validation

The marker definitions in this educational prototype should not be treated as a validated clinical variant database.

---

# Future Improvements

The project can be extended in several directions.

## 1. Real FASTA Input

Allow users to provide:

```text
patient.fasta
```

instead of generating synthetic sequences.

Pipeline:

```text
FASTA File
   ↓
Sequence Parser
   ↓
DNA String
   ↓
Aho-Corasick
   ↓
Detected Markers
```

## 2. Larger Marker Database

The marker panel could be moved from hardcoded Java objects to:

```text
CSV
JSON
SQL database
```

For example:

```text
marker_id
gene
disease
chromosome
position
reference
alternate
mutation_type
```

## 3. Approximate Matching

Real sequencing data may contain errors.

Future versions could incorporate:

* edit distance
* Hamming distance
* alignment algorithms
* k-mer based matching

## 4. FASTQ Support

Instead of only FASTA, the system could process FASTQ files containing sequencing-quality scores.

## 5. Web Interface

A frontend could allow a user to:

```text
Upload DNA file
       ↓
Select marker panel
       ↓
Run screening
       ↓
View detected markers
       ↓
Generate report
```

## 6. Database Integration

Marker metadata could be stored in a database and retrieved dynamically.

## 7. Security

A real patient-data system would require appropriate:

* Authentication
* Authorization
* Encryption
* Audit logging
* Secure storage
* Data retention policies
* Privacy controls

---

# Algorithm Comparison

| Feature                   | Aho-Corasick                      | KMP                     |
| ------------------------- | --------------------------------- | ----------------------- |
| Primary purpose           | Multiple-pattern matching         | Single-pattern matching |
| DNA sample scan           | Shared scan                       | Repeated per marker     |
| Preprocessing             | Trie + failure links              | LPS table               |
| Multiple markers          | Efficient                         | Repeated searches       |
| Individual pattern search | Yes                               | Yes                     |
| Overlapping matches       | Supported                         | Supported               |
| Main use in project       | Primary algorithm                 | Baseline                |
| Search complexity         | O(n + matches) after construction | O(n + m) per pattern    |

Here:

* `n` = DNA sample length
* `m` = marker length
* `matches` = number of detected pattern occurrences

For a marker panel containing `k` patterns, KMP performs a separate search for each pattern, whereas Aho-Corasick processes the panel together.

---

# Example Workflow

Consider a synthetic sample:

```text
sample_sickle_cell.fasta
```

The system performs:

```text
1. Read FASTA
       ↓
2. Extract nucleotide sequence
       ↓
3. Load MarkerPanel
       ↓
4. Build Aho-Corasick automaton
       ↓
5. Scan DNA
       ↓
6. Find HBB_HbS
       ↓
7. Record position
       ↓
8. Retrieve marker metadata
       ↓
9. Print structured result
       ↓
10. Compare against KMP
```

---

# Expected Demonstration

The four synthetic samples are designed to demonstrate different marker combinations:

| Sample                    | Intended synthetic content        |
| ------------------------- | --------------------------------- |
| `HBB_reference`           | Reference sequence                |
| `sample_sickle_cell`      | Sickle-cell teaching variant      |
| `sample_beta_thalassemia` | Beta-thalassemia teaching variant |
| `sample_combined`         | Both teaching variants            |

The exact output should be determined by running the program rather than assuming the displayed positions or timings.

---

# Educational Purpose

This project primarily demonstrates the application of **Data Structures and Algorithms to computational biology**.

The central idea is:

> Convert DNA marker screening into a multiple-pattern string-search problem and use Aho-Corasick to efficiently search a DNA sequence against a marker panel.

It connects:

```text
Data Structures
       +
String Algorithms
       +
DNA Sequences
       +
Marker Metadata
       ↓
Computational DNA Screening Prototype
```

---

# Authors / Project Information

**Project:** DNA Marker Screening System

**Language:** Java

**Primary Algorithm:** Aho-Corasick

**Baseline Algorithm:** Knuth-Morris-Pratt (KMP)

**Data:** Synthetic DNA sequences

**Domain:** Computational Biology / Bioinformatics / Data Structures & Algorithms

---

# License / Usage

This project is intended for **educational and research-prototype purposes**.

The marker definitions and generated DNA sequences are not intended for clinical diagnosis or medical decision-making.
