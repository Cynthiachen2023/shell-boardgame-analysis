## Board Game Dataset: Bash Script Guide

This project contains three Bash scripts to check, clean, and analyze a [Board Game Geek (BGG) dataset](https://www.kaggle.com/datasets/andrewmvd/board-games). They answer four research questions and meet the CITS4407 Assignment 2 requirements.

---

### Script 1: `empty_cells`

**Purpose:** Count the number of empty cells in each column.

**Usage**

```bash
./empty_cells <file_path>
```

**Details**

* Assumes the first row is a header.
* Fields are separated by `;`.
* Counts empty (zero‐length) values per column.
* Input file must be non‐empty and end in `.txt`.

#### Example

```text
$ ./empty_cells bgg_dataset.txt

/ID: 16
Name: 0
Year Published: 1
Min Players: 0
Max Players: 0
Play Time: 0
Min Age: 0
Users Rated: 0
Rating Average: 0
BGG Rank: 0
Complexity Average: 0
Owned Users: 23
Mechanics: 1598
Domains: 10159
```

---

### Script 2: `preprocess`

**Purpose:** Clean and normalize the raw data file by performing these transformations and writing the result to **standard output**.

**Usage**

```bash
./preprocess <file_path>
```

You can redirect its output if you like:

```bash
./preprocess sample.txt > cleaned_output
```

**Cleaning Steps**

1. Replace field separator `;` with a tab (`\t`).
2. Convert Windows line endings (`\r\n`) to Unix (`\n`).
3. Change commas in numeric fields to dots.
4. Remove all non-ASCII characters.
5. Auto-fill missing `/ID` values by incrementing from the current maximum ID.
6. Write the cleaned lines to **stdout** in the same order.

#### Example

```text
$ ./preprocess tiny_sample.txt

/ID     Name    Year Published  Min Players     Max Players     Play Time       Min Age Users Rated Rating Average  BGG Rank        Complexity Average      Owned Users     Mechanics  Domains
174430  Gloomhaven      2017    1       4       120     14      42055   8.79    1       3.8668323   Action Queue, Action Retrieval, Campaign / Battle Card Driven, Card Play Conflict Resolution, Communication Limits, Cooperative Game, Deck Construction, Deck Bag and Pool Building, Grid Movement, Hand Management, Hexagon Grid, Legacy Game, Modular Board, Once-Per-Game Abilities, Scenario / Mission / Campaign Game, Simultaneous Action Selection, Solo / Solitaire Game, Storytelling, Variable Player Powers  Strategy Games, Thematic Games
....
```

---

### Script 3: `analysis`

**Purpose:** Use the cleaned data (as produced by `preprocess`) to answer four research questions.

**Usage**

```bash
./analysis <cleaned_file>
```

**Questions Answered**

1. Which game **mechanic** is most popular?
2. Which game **domain** (type) is most common?
3. What is the Pearson correlation between **year published** and **average rating**?
4. What is the Pearson correlation between **complexity** and **average rating**?

**Implementation Highlights**

* Uses `gawk` for data aggregation and flow control.
* Functions:

  * `pearson_variables(x, y, name)`: gather sums for correlation.
  * `pearson_r(name)`: compute Pearson’s *r*, with division-by-zero safety.
  * `find_max(array)`: find the key with the highest count.
* Clears all global arrays at `BEGIN` to avoid state leftover.
* Handles irregular whitespace and missing data.

#### Example

```text
$ ./preprocess sample.txt > sample1_actual.tsv
$ ./analysis sample1_actual.tsv

The most popular game mechanics is Hexagon Grid found in 2 games
The most style of game is Wargames found in 2 games
The correlation between the year of publication and the average rating is -0.295
The correlation between the complexity of a game and its average rating is 0.824
```

---

## Testing Suggestions

In the UniApps Docker image, run:

```bash
./empty_cells sample.txt
./preprocess sample.txt > sample_clean
./analysis sample_clean
```

Compare results with expected outputs, e.g.:

```bash
diff sample.tsv sample_clean
diff sample_expected_analysis.txt <(./analysis sample_clean)
```

You can also test edge cases (e.g. a one-line file) to ensure robustness.
