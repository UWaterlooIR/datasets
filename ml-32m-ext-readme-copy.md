
# ML-32M-Extension

This repository contains the resource described in "Extending
MovieLens-32M to Provide New Evaluation Objectives" by Smucker and
Chamani.

## Reminder about Restrictions on Usage

This dataset is **only available to researchers** for **research on
evaluation of recommendation systems and similar purposes**.

To access the dataset, researchers must:

- Submit a form attesting to their status as a researcher.
- Agree to keep the dataset private and not publicly share it.
- Use the dataset only for research on evaluation of recommendation systems and similar purposes.
- Refrain from attempting to identify any anonymous participants in the dataset.

Any other uses of the dataset are strictly prohibited.  These
restrictions are in-place to abide by the informed consent that
participants provided for "broad consent for the storage and future
unspecified use of data" as per [TCPS 2 (2022), Chapter 3, Section
E](https://ethics.gc.ca/eng/tcps2-eptc2_2022_chapter3-chapitre3.html#e).
To have the data available without restriction would require blanket
consent, which is not permitted by TCPS 2 (2022).

## Contents of Repository

This repository contains a combination of data and scripts to generate
files and document the creation of certain files.  As described in the paper, there
were two phases to the study: Phase 1 (P1) and Phase 2 (P2).  This respository
only contains the data for the 51 participants that completed P2.

The data files:

+ p2-movielens-profiles.csv : Contains the MovieLens ratings of the 51 participants that they submitted to us at the start of the study. These are the ratings they downloaded from movielens.org and sent to us.
+ p2-ratings.csv : Contains the relevance judgements for the pools as well as the consistency check movies.
+ database-movies.csv : Contains an export of the information we obtained from TMDB to display to the participants for judging movies.
+ explicit-ratings-csv.patch : A patch file that is used by our scripts to create the explicit-ratings.csv, which once built contains the tranformed and extended version of ML-32M.
+ qrels/*.qrels : The qrels files.

The scripts:

+ make.sh : Please read below before using.  Used to generated explicit-ratings.csv, implicit-ratings.csv, and filtered-movies.csv
+ append-p2-profiles-to-explicit-ratings.py : Used by make.sh to extend our transformed ML-32M to include the participant's profiles.
+ explicit-to-implicit.py : Used by make.sh and shows how we create implicit-ratings.csv.
+ p2-ratings-to-qrels.py : Used to generate the qrels files.  Primary purpose is to document the qrels files, for the files are already generated and in `qrels/`.

## Building the ML-32M-Extension

1. Obtain a copy of ML-32: https://grouplens.org/datasets/movielens/32m/
2. Edit make.sh and edit this line:
    ```
    ML32M_DIR="/share/corpora/movielens/ml-32m"  # Change this to where you have ml-32m stored
    ```
    so that ML32M_DIR is the full path to your ml-32m location.

3. Execute: bash make.sh 

Once finished, you should have explicit-ratings.csv, implicit-ratings.csv, and filtered-movies.csv.

#### Converting MovieLens format files to RecBole format

To make a dataset for Recbole, follow the directions for ml-20m at:

https://github.com/RUCAIBox/RecSysDatasets/blob/master/conversion_tools/usage/MovieLens.md

but use the implicit-ratings.csv as ratings.csv and
filtered-movies.csv as movies.csv inside your fake ml-20m dataset.

For example, if I've copied implicit-ratings.csv and
filtered-movies.csv to a directory ml-implicit as ratings.csv and
movies.csv, respectively, then you can convert to Recbole atomic format
following their directions as follows:

python run.py --dataset ml-20m --input_path ml-implicit --output_path output_data/ml-implicit --convert_inter --convert_item

## Details about the Files

### p2-movielens-profiles.csv

This file contains the ratings that the 51 participants provided to us
at the start of the study.  Each participant was given instructions on
how to dowload their ratings from movielens.org and send to us.

Each participant has been assigned a random participant_id.  We also
assigned to the participant a dataset_id, which is used as the user_id
when we append their profile ratings to explicit-ratings.csv.  The
dataset_id = participant_id + 200948.  The id 200948 is the maximum
user_id in ML-32M ratings.csv.

Please note that the participant_id values are not contigious and
contain gaps.  These gaps are the result of removal of participants
who did not finish P2.

The movie_id column is a MovieLens movie_id, and the movielens_rating
is the rating.

### p2-ratings.csv

This file contains the relevance judgments made by the 51
participants.  Each row represents the assessment of a movie made by a
participant, capturing their familiarity with the movie,
interest-in-watching, and other relevant details.

#### Columns and Descriptions

| Column Name        | Data Type  | Description |
|--------------------|-----------|-------------|
| `participant_id`   | Integer   | A randomly assigned participant ID starting at 1. |
| `dataset_id`      | Integer   | A unique dataset ID, computed as `200948 + participant_id` to match profile appended to explicit-ratings.csv |
| `movie_id`        | Integer   | A positive integer representing the MovieLens movie ID. |
| `seen_status`     | Boolean (`t`/`f`) | Indicates whether the participant has seen the movie (`t` for true, `f` for false). |
| `familiarity`     | String    | Participant's familiarity with the movie: \{ `Never` \| `Familiar` \| `Very familiar` \| `Seen` \}. |
| `p1_check`        | Boolean   | Indicates if the movie was part of phase 1 (P1) consistency check. `P1` for true, empty for false. |
| `p2_check`        | Boolean   | Indicates if the movie was part of phase 2 (P2) consistency check. `P2` for true, empty for false. |
| `interest_level`  | String    | Participant's stated interest level in the movie: \{ `Not interested` \| `Somewhat interested` \| `Interested` \| `Very interested` \| `Extremely interested` \}. |
| `rating`          | Float     | The participant's rating of the movie, ranging from `0.5` to `5.0` in increments of `0.5`. This is a predicted rating if the movie was not seen, and a recalled rating if the movie was seen. |
| `rank_p2`         | Integer (Nullable) | Rank assigned in phase 2, if available \(`1`, `2`, or `3`\). Empty values indicate no ranking was assigned. |
| `movielens_rating`| Float | The original MovieLens rating from p2-movielens-profiles.csv for consistency check movies only, **not used for qrels**. |

**Note that for 2 rows, the `familiarity` is missing.**

See p2-ratings-to-qrels.py for an example of how to process p2-ratings.csv.

### explicit-ratings.csv, implicit-ratings.csv, filtered-movies.csv

These are MovieLens format files in the format of ratings.csv and
movies.csv.  The explicit-ratings.csv file contains the filtered and
transformed version of ML-32M described in the paper as well as the
appended 51 participant profiles (user_id > 200948).  The
implicit-ratings.csv file is created from the explicit-ratings.csv by
only keeping ratings >= 4.0 and users with at least 5 ratings.  The
filtered-movies.csv represents a movies.csv file with only the movies
in explicit-ratings.csv.

Please note that all participant ratings are appended with the same
timestamp.  We did not obtain the data from the participants that
captures the time of their ratings.  We've set the time of their
ratings to fall later than the creation date of ML-32M as explained in
append-p2-profiles-to-explicit-ratings.py.  Each participant rating
gets the timestamp 1697216400, which is Friday October 13, 2023
12:00:00 (pm) in time zone America/Chicago (CDT).

### qrels/*.qrels

The paper explains how to use these files and what they contain.  To see
how they were created, see p2-ratings-to-qrels.py.

These are in standard TREC qrels format:

+ column 1: topic_id, which in our case is the dataset_id for a participant
+ column 2: ignore the Q0
+ column 3: movie_id
+ column 4: a preference level for the `*interest*.qrels` preference-based qrels, a rating for the `*rating*.qrels` files.

#### Resources to use with qrels files

The *compatibility* measure:

+ Version used in paper: https://github.com/trec-health-misinfo/Compatibility 
+ Official version: https://github.com/claclark/Compatibility

trec_eval for nDCG and many other measures: https://github.com/usnistgov/trec_eval






