# Super Mario Structure Dataset

Section-level structure annotations of 554 Super Mario pieces, from the piano and ensemble transcriptions of NinSheetMusic. Every piece has a functional and thematic segmentation made with Gemini 3.1 Pro. For 49 pieces, two human annotators also made independent hierarchical segmentations. Piece metadata and bar maps link the annotations to the scores, which users download themselves: the scores are third-party transcriptions and are not redistributed here.

The annotations can serve music structure analysis, segmentation and repetition studies, the evaluation of automatic or language-model annotation against human annotators, and structure-aware music generation. The folder `simllama/` holds the data of one such use: section pairs with similarity labels and a listening test, from the SimLlama paper (see [SimLlama data](#simllama-data)).

## Contents

| File or Folder | Content | Count | Format |
|---|---|---|---|
| `metadata.csv` | Piece metadata and exclusion flags | 554 rows | CSV |
| `annotations/gemini_prompt_v1/` | Gemini structure annotations | 554 files | JSON |
| `annotations/human/pieces.csv` | Index of the human-annotated pieces | 49 rows | CSV |
| `annotations/human/annotator1/` | Hierarchical annotations from Annotator 1 | 49 files | CSV |
| `annotations/human/annotator2/` | Hierarchical annotations from Annotator 2 | 49 files | CSV |
| `bar_maps.csv` | Which score each piece used, and the bar order of hand-unrolled scores | 334 rows | CSV |
| `simllama/pairs.csv` | Directed section pairs with similarity labels | 3,304 rows | CSV |
| `simllama/listening_test/answer_key.csv` | Trial design and reference measures | 32 rows | CSV |
| `simllama/listening_test/ratings.csv` | Anonymised listener ratings | 26 rows | CSV |
| `simllama/listening_test/clips/` | Generated audio clips | 60 files | MP3 |

**Identifiers.** A piece's integer `id` is written with five digits in file names: annotation `00410.json`, human annotation `00410.csv`, section names `00410_s1_B_9-16`. The NinSheetMusic sheet id of a piece is in `annotations/human/pieces.csv` and in the `note` column of `bar_maps.csv`.

## Annotations

### Gemini annotations: `annotations/gemini_prompt_v1/*.json`

Gemini 3.1 Pro annotated each score from ABC notation, with the prompt `scripts/annotation/prompt_v1.md` of the code repository. Each annotation has two levels: functional sections, and theme sections within them. The 554 files hold 1,917 theme sections.

- `Tile` (sic): title of the piece.
- `IsAdaptive`: true if the piece changes with game state.
- `IsStinger`: true for a short event cue.
- `Function`: the functional sections. Each item has:
  - `BarRange`: first and last bar, inclusive.
  - `Function`: `In` intro, `Lp` loop body (repeats while the game plays), `Ln` linear body (a main body that does not loop), `Br` bridge between main sections, `Ou` outro, `St` stinger.
  - `Inference`: the model's reason, in free text.
- `Section`: the theme sections. Each item has:
  - `BarRange`: first and last bar, inclusive.
  - `Function`: the functional section it lies in.
  - `Section`: a theme letter (`A`, `B`, `C` ...). Sections with the same letter in a piece hold the same material. `X` marks a section without a theme: every intro, outro, bridge and stinger, and non-thematic passages in a body.
  - `Inference`: the model's reason, in free text.

### Human annotations: `annotations/human/`

Two annotators segmented 49 pieces, drawn at random across piece lengths. They worked independently and saw no Gemini annotation. Each file has one row per lowest-level segment:

- `start_bar`: the first bar of the segment.
- `length`: the segment's length in bars.
- `L1_letter` to `L5_letter`: the segment's letter at each of five nested levels. Within a level, segments with the same letter hold related material.
- `L1_function` to `L5_function`: the segment's role in its group: `it` initiation (states the idea), `mi` middle (develops or contrasts), `en` ending (closes), `pr` preparation (leads into a segment), `po` postlude (settles after one), `tr` transition (links two), `si` silence (letter `x`).
- `comment`: free text by the annotator.

L1 is the finest level; each higher level groups consecutive segments of the level below. Bars count the score with repeats written out; some files start at bar 0, a pickup.

`annotations/human/pieces.csv` joins the files: `local_id` (the five-digit piece id), `sheet_id` (NinSheetMusic), `title`, `in_paper_pairs` (1 if the piece is in `simllama/pairs.csv`; 37 are), `comparable` (0 for two pieces whose human and Gemini bar counts differ, because one side wrote out repeats), `note`.

### Agreement

Gemini agrees with each annotator about as well as the two annotators agree with each other. The comparison covers the 47 comparable pieces and only main-body sections, since the two schemes label intros, endings and silence differently.

- **Same material.** Two Gemini sections hold the same material for an annotator when the annotator's letters for them agree. Both annotators judge 128 such pairs of sections. Cohen's kappa, agreement beyond chance, with 95% intervals from resampling whole pieces:

  | Between | Kappa | 95% interval |
  |---|---:|---|
  | Gemini and Annotator 1 | 0.59 | 0.25–0.81 |
  | Gemini and Annotator 2 | 0.68 | 0.44–0.84 |
  | Annotator 1 and Annotator 2 | 0.58 | 0.35–0.76 |

- **Boundaries.** A Gemini section is consistent with an annotator when it equals one human segment, or a run of consecutive segments, at some level. Of the 177 Gemini sections, 87% are consistent with Annotator 1 and 82% with Annotator 2.

The intervals are wide: the data fit parity between Gemini and the annotators, not better or worse agreement. The Gemini annotations of the other pieces are unchecked and contain errors.

## Metadata: `metadata.csv`

- `id`: integer piece identifier, as in file names and `piece_id` elsewhere.
- `original_filename`: title of the score on NinSheetMusic, as downloaded.
- `num_voices`: number of voices or tracks.
- `total_bars`: bar count as transcribed.
- `time_signatures`, `key_signatures`: signatures found in the score.
- `isStinger`: true for short stinger cues.
- `instruments`: instrument labels found in the score.
- `is_double_piano`: true for double-piano transcriptions.
- `is_mixed_instr`: true for mixed-instrument arrangements.
- `excluded`: true for double-piano and mixed-instrument pieces, which `simllama/pairs.csv` leaves out.
- `fix_status`: 1 if the score was edited by hand before annotation (28 pieces), empty otherwise; `bar_maps.csv` describes the edit for the pieces it covers.

## Scores and Bar Maps

Download the scores from https://www.ninsheetmusic.org. Most pieces were annotated as downloaded. For some, repeats and multiple endings were first written out by hand, and the annotations follow that unrolled score. `bar_maps.csv` records which score each of the 334 pieces in `simllama/pairs.csv` used:

| `source` | Pieces | Rebuild from the download |
|---|---:|---|
| `raw` | 310 | Use the score as downloaded |
| `fix`, `verified = yes` | 7 | Exact: apply `order` with `scripts/data/bar_maps.py apply` of the code repository |
| `fix`, `verified = no` | 15 | Not exact: the hand edit also changed notes or rests (`note` gives the first differing bar); most have no `order` |
| `unknown` | 2 | The bar count matches no available score (`00282`, `00543`) |

The 15 approximate pieces are `00003 00058 00189 00190 00201 00223 00281 00327 00340 00341 00415 00469 00505 00536 00544`.

Columns:

- `piece_id`: the piece.
- `source`: the score used (`raw`, `fix`, `unknown`; the script can also report `raw_played` and `fix_played`, the played-through order of the repeats, which no piece needed).
- `fix_date`: last change of the hand-unrolled file.
- `bars_raw`: bar count of the downloaded score.
- `bars_used`: bar count of the score used.
- `order`: the unrolled bar sequence as ranges of original bar numbers (e.g. `1-4,5-12,5-12,13-20`); bar 0 is a pickup.
- `verified`: whether applying `order` to the download reproduces the score used.
- `note`: remarks on repeats or hand edits, and the NinSheetMusic sheet id.

The maps hold bar numbers only, no note content.

## SimLlama Data

`simllama/` holds the data of the paper "How Close to the Theme? Similarity-Label Conditioning for Game Music Generation" (Luo, Guo, Shi, Reiss; submitted to Computer Music Journal). The paper trains SimLlama, a model that generates a section at a requested similarity to a given section. Code and model: https://github.com/ShxLuo-Saxon/simllama.

### `simllama/pairs.csv`

3,304 directed pairs of sections within the same piece, across 334 pieces. Sections are the Gemini theme sections. Splits: 2,398 train, 478 validation, 428 test. Labels: 1,090 `sim:high`, 1,172 `sim:mid`, 1,042 `sim:low`.

- `pair_id`: unique pair identifier.
- `piece_id`: the piece.
- `split`: `train`, `val` or `test`.
- `context_file`: the context section: piece, section index, theme letter and bar range, with the extension of the section array file (for example `00410_s1_B_9-16.npy`). The arrays are derived from the scores and are not released.
- `target_file`: the target section, in the same form.
- `context_ntokens`, `target_ntokens`: note counts of the two sections.
- `compound_ks`: compound similarity (CS) between context and target, from 0 to 1. It sets the label: `sim:high` at 0.78 and above, `sim:low` below 0.60.
- `sim_bucket`: the label (`sim:high`, `sim:mid`, `sim:low`).
- `tgt_key_shift`: the transposition of the target, 0 to 11 semitones, that best matches the context's pitch classes (chroma cosine); 0 when transposing gains less than 0.1.

### Listening test

The test asked listeners how similar generated clips are to a reference section.

- 32 trials: 30 test trials and 2 attention checks.
- In a test trial, listeners heard a reference context section and two clips generated from it, A and B: one with `sim:high`, the other with a lower label.
- In a check trial, A and B were real excerpts: the true continuation and a section of an unrelated piece. Check trials are not released as audio.
- Listeners rated each clip's similarity to the reference from 0 to 100.

**Clips.** `simllama/listening_test/clips/` holds the 60 generated clips of the 30 test trials, `TrialNN_2_A.mp3` and `TrialNN_3_B.mp3`, each about 10 seconds of solo piano. The reference clips derive from the NinSheetMusic scores and are not released; users who rebuild the scores can render them.

**`answer_key.csv`**, one row per trial:

- `trial`: `Trial01` to `Trial32`, in the order listeners heard them.
- `kind`: `test` or `check`.
- `stratum`: the sampling stratum, by the sign of `delta`.
- `ctx_stem`: the context section, as in `pairs.csv`.
- `delta`: `cs_high` minus `cs_low`.
- `cs_high`, `cs_low`: CS of the `sim:high` clip and of the other clip to the context.
- `high_slot`: the clip, `A` or `B`, generated with `sim:high`.
- `other_is`: the label of the other clip; for checks, what the foil was.
- `ctx_sec`: length of the context clip in seconds.

**`ratings.csv`**: 26 listeners, 66 columns. Each listener's study ID was replaced with a code, `L01` to `L26`, in column `listener`; the table linking codes to IDs was not kept. The file keeps the ratings and the answer to the question on formal music education, in `music_education`: `none` (2), `up to 3 years` (14), `4 to 10 years` (9), `over 10 years` (1). Timestamps, location, IP address, browser data and incomplete sessions were removed. The rating columns keep the names of the questionnaire export (`N.Similarity of clip A to the reference (0-100):A`, where N is the trial number), which the analysis scripts read.

## Collection

The pieces are the Super Mario transcriptions on NinSheetMusic, a fan site of piano and ensemble arrangements. Gemini annotated all 554 pieces; two human annotators segmented 49 of them. The listening test was approved by the Queen Mary University of London research ethics committee (reference DSEECS26.094). Unpaid volunteers, invited by email, took it online and consented to the publication of anonymised ratings. The work was supported by the Queen Mary University of London and China Scholarship Council (CSC) Joint Scholarship.

## Uses

- **Suitable for:** music structure analysis and segmentation of game music; studies of repetition and form; evaluating automatic or language-model structure annotation against human annotators; structure-aware or similarity-controlled generation, together with the scores, which users obtain themselves; listening-test analyses.
- **Not suitable for:** redistributing the music (no score content is included); claims beyond piano transcriptions of one game series. The Gemini annotations are machine-made and contain errors; only 49 pieces have human annotations.

## Distribution and Maintenance

- Version 1.0, archived on Zenodo: [doi:10.5281/zenodo.23915907](https://doi.org/10.5281/zenodo.23915907). The DOI [10.5281/zenodo.23915906](https://doi.org/10.5281/zenodo.23915906) always resolves to the latest version.
- Errors and questions: open an issue in this repository, or write to Shangxuan Luo (s.luo@qmul.ac.uk). Corrections are released as new versions; earlier versions stay available on Zenodo.

## Licence

All files are released under the Creative Commons Attribution 4.0 International licence (CC BY 4.0).

## Citation

Please cite the dataset:

```bibtex
@dataset{luo2026supermario,
  title={Super Mario Structure Dataset},
  author={Luo, Shangxuan and Guo, Zixun and Shi, Jinjie and Reiss, Joshua},
  year={2026},
  version={1.0},
  publisher={Zenodo},
  doi={10.5281/zenodo.23915907}
}
```

If you use the `simllama/` data or the agreement figures, please also cite the paper:

```bibtex
@misc{luo2026simllama,
  title={How Close to the Theme? Similarity-Label Conditioning for Game Music Generation},
  author={Luo, Shangxuan and Guo, Zixun and Shi, Jinjie and Reiss, Joshua},
  year={2026},
  eprint={2611.01047},
  archivePrefix={arXiv},
  note={Submitted to Computer Music Journal}
}
```
