# ToTTo Dataset

ToTTo is an open-domain English table-to-text dataset with over 120,000 training examples that proposes a controlled generation task: given a Wikipedia table and a set of highlighted table cells, produce a one-sentence description. 

During the dataset creation process, tables from English Wikipedia are matched with (noisy) descriptions. Each table cell mentioned in the description is highlighted and the descriptions are iteratively cleaned and corrected to faithfully reflect the content of the highlighted cells.

We hope this dataset can serve as a useful research benchmark for high-precision conditional text generation. 

You can find more details, analyses, and baseline results in [our paper](https://arxiv.org/abs/2004.14373). You can cite it as follows:

```
@article{parikh2020totto,
  title={ToTTo: A Controlled Table-To-Text Generation Dataset},
  author={Parikh, Ankur P and Wang, Xuezhi and Gehrmann, Sebastian and Faruqui, Manaal and Dhingra, Bhuwan and Yang, Diyi and Das, Dipanjan},
  journal={arXiv preprint arXiv:2004.14373},
  year={2020}
```

## Getting Started
**Download the ToTTo data**

The ToTTo dataset is released under the [Creative Commons Share-Alike 3.0](https://creativecommons.org/licenses/by-sa/3.0/) license.

To download the data from the command line:
```
 wget https://storage.googleapis.com/totto/totto_data.zip
 unzip totto_data.zip
```
(or alternatively copy the above url into your browser address bar.)

Inside the `totto_data` directory you should see three files: `totto_train_data.jsonl`, `totto_dev_data.jsonl`, and `unlabeled_totto_test_data.jsonl` for the training, development, and unlabeled test sets respectively.

**Download the evaluation scripts**

You can find evaluation scripts and some exploratory processing scripts at [this repository](https://github.com/google-research/language/tree/master/language/totto). It also includes a separate README file with instruction on how to run the evaluation. 

## Dataset Description

The ToTTo dataset consists of three `.jsonl` files, where each line is a JSON dictionary with the following format:

```json
{
  "table_page_title": "'Weird Al' Yankovic",
  "table_webpage_url": "https://en.wikipedia.org/wiki/%22Weird_Al%22_Yankovic",
  "table_section_title": "Television",
  "table_section_text": "",
  "table": "[Described below]",
  "highlighted_cells": [[22, 2], [22, 3], [22, 0], [22, 1], [23, 3], [23, 1], [23, 0]],
  "example_id": 12345678912345678912,
  "sentence_annotations": [{"original_sentence": "In 2016, Al appeared in 2 episodes of BoJack Horseman as Mr. Peanutbutter's brother, Captain Peanutbutter, and was hired to voice the lead role in the 2016 Disney XD series Milo Murphy's Law.",
                  "sentence_after_deletion": "In 2016, Al appeared in 2 episodes of BoJack Horseman as Captain Peanutbutter, and was hired to the lead role in the 2016 series Milo Murphy's Law.",
                  "sentence_after_ambiguity": "In 2016, Al appeared in 2 episodes of BoJack Horseman as Captain Peanutbutter, and was hired for the lead role in the 2016 series Milo Murphy's 'Law.",
                  "final_sentence": "In 2016, Al appeared in 2 episodes of BoJack Horseman as Captain Peanutbutter and was hired for the lead role in the 2016 series Milo Murphy's Law."}],
}
```

The `table` field is a `List[List[Dict]]`. The outer lists represents rows and the inner lists columns. Each `Dict` has the fields `column_span: int`, `is_header: bool`, `row_span: int`, and `value: str`. The first two rows for the example above look as follows:

```json
[
  [
    {    "column_span": 1,
         "is_header": true,
         "row_span": 1,
         "value": "Year"},
    {    "column_span": 1,
         "is_header": true,
         "row_span": 1,
         "value": "Title"},
    {    "column_span": 1,
         "is_header": true,
         "row_span": 1,
         "value": "Role"},
    {    "column_span": 1,
         "is_header": true,
         "row_span": 1,
         "value": "Notes"}
  ],
  [
    {    "column_span": 1,
         "is_header": false,
         "row_span": 1,
         "value": "1997"},
    {    "column_span": 1,
         "is_header": false,
         "row_span": 1,
         "value": "Eek! The Cat"},
    {    "column_span": 1,
         "is_header": false,
         "row_span": 1,
         "value": "Himself"},
    {    "column_span": 1,
         "is_header": false,
         "row_span": 1,
         "value": "Episode: 'The FugEektive'"}
  ], ...
]
```

-The table metadata consists of the `table_page_title`, `table_section_title`, and `table_section_text` strings to help give the model more context about the table.

-The `highlighted_cells` field is a `List[[row_index, column_index]]` where each `[row_index, column_index]` pair indicates that `table[row_index][column_index]` is highlighted.

-The `example_id` is simply a unique id for this example.

-The `sentence_annotations` field consists of the `original sentence` and the sequence of revised sentences performed in order to produce the `final_sentence`. See our [paper](https://arxiv.org/abs/2004.14373) for more details.


To help understand the dataset, you can find a sample of the train and dev sets in the `sample/` folder of our [supplementary repository](https://github.com/google-research/language/tree/master/language/totto). It additionally provides the `create_table_to_text_html.py` script that visualizes examples, the output of which you can also find in the `sample/` folder.

### Official Task

The official task described in our [paper](https://arxiv.org/abs/2004.14373) is given the `table`, `highlighted cells`, and table metadata (`table_page_title`, `table_section_title`, and `table_section_text`) as input, to generate the `final_sentence`.

### Dev and Test Set

The dev and test set have between two and three references for each example, which are added to the list at the `sentence_annotations` key. The test set annotations are *private* and thus not included in the data. 

If you want us to evaluate your model on the development or the private test set, please submit your files [here](https://forms.gle/AcF9TRqWrPhPzztt7). You can find more submission information [below](https://github.com/google-research-datasets/ToTTo#leaderboard-submission). By emailing us or by submitting prediction files, you consent to being contacted by Google about your submission, this dataset or any related competitions.

We provide two splits within the dev and test sets - one uses previously seen combinations of table headers and one uses unseen combinations. The sets are marked using the `overlap_subset: bool` flag that is added to the JSON representation. By filtering the evaluation to examples with the flag set to `true`, you will be able to test the generalization ability of your model.

****

# Leaderboard

We are maintaining a leaderboard with official results on our test set.

The leaderboard indicates whether or not a model was trained on any auxiliary Wikipedia data. This is because our tables and (unrevised) test targets are from Wikipedia and thus we would like to study the effect of using additional Wikipedia data to train models.

We ask you to **not** incorporate any part of the ToTTo development set into the training data, and only use it for validation/hyperparameter tuning as development sets are typically used.

<table>
  <tr>
    <th></th>
    <th></th>
    <th></th>
    <th></th>
    <th colspan="2">Overall</th>
    <th colspan="2">Overlap Subset</th>
    <th colspan="2">Non-Overlap Subset</th>
  </tr>
  <tr>
    <th></th>
    <th>Model</th>
    <th>Link</th>
    <th>Uses Wiki</th>
    <th>BLEU</th>
    <th>PARENT</th>
    <th>BLEU</th>
    <th>PARENT</th>
    <th>BLEU</th>
    <th>PARENT</th>
  </tr>
    <tr>
    <td>1.</td>
      <td> maninblack (<i>Google</i>) </td>
      <td> - </a></td>
    <td>yes</td>
    <td><b>49.5</b></td>
    <td><b>58.4</b></td>
    <td><b>57.5</b></td>
    <td><b>62.6</b></td>
    <td><b>41.4</b></td>
    <td><b>54.2</b></td>
  </tr>
  <tr>
    <td>2.</td>
    <td>BERT-to-BERT (Wiki+Books)</td>
    <td><a href="https://arxiv.org/abs/1907.12461">[Rothe et al., 2019]</a></td>
    <td>yes</td>
    <td>44.0</td>
    <td>52.6</td>
    <td>52.7</td>
    <td>58.4</td>
    <td>35.1</td>
    <td>46.8</td>
  </tr>
  <tr>
    <td>3.</td>
    <td>BERT-to-BERT (Books)</td>
    <td><a href="https://arxiv.org/abs/1907.12461">[Rothe et al., 2019]</a></td>
    <td>no</td>
    <td>43.9</td>
    <td>52.6</td>
    <td>52.7</td>
    <td>58.4</td>
    <td>34.8</td>
    <td>46.7</td>
  </tr>
  <tr>
    <td>4.</td>
    <td>Pointer Generator</td>
    <td><a href="https://www.aclweb.org/anthology/P17-1099/">[See et al., 2017]</a></td>
    <td>no</td>
    <td>41.6</td>
    <td>51.6</td>
    <td>50.6</td>
    <td>58.0</td>
    <td>32.2</td>
    <td>45.2</td>
  </tr>
  <tr>
    <td>5.</td>
    <td>Content Planner</td>
    <td><a href="https://www.aaai.org/ojs/index.php/AAAI/article/view/4668">[Puduppully et al., 2019]</a></td>
    <td>no</td>
    <td>19.2</td>
    <td>29.2</td>
    <td>24.5</td>
    <td>32.5</td>
    <td>13.9</td>
    <td>25.8</td>
  </tr>
</table>

## Leaderboard Submission

If you want to submit dev and test outputs, please format your predictions as a single `.txt` file with line-separated predictions. The predictions should be in the same order as the examples in the `test.jsonl` file.
You can upload your prediction files [here](https://forms.gle/AcF9TRqWrPhPzztt7). If you run into any issues, you can contact us at totto@google.com. By emailing us or by submitting prediction files, you consent to being contacted by Google about your submission, this dataset or any related competitions.

