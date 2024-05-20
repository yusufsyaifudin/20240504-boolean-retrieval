# 2024-05-04 Information Retrieval - Boolean Retrieval

## Code

All code in this report can be obtained through public repository: [https://github.com/yusufsyaifudin/20240504-boolean-retrieval](https://github.com/yusufsyaifudin/20240504-boolean-retrieval)


## System Architecture

This task is to create inverted index based on the document collections (term based).
For this, I create two task: Indexing and Retrieval.

For Indexing task, I separate into two sub-task: data cleaning and create the index.
For Retrieval task, I create function to the index model and try two query for each boolean logical operator: `AND`, `OR`, and `NOT`.

### Create Inverted Index

The process is as follow:
1. Open file CSV.
2. For each row, I get the `content` value.
3. For each `content` value, I do case-folding (transform all characters into lower case).
4. Then I split by space to get `token` (or word).
5. Each `token` is streamed into functions: `remove_tweet_special`, `remove_number`, `remove_punctuation`, `remove_whitespace_multiple`, `remove_single_char` to clean unnecessary character.
   If after this process the `token` text become empty string, we skip this `token` for further process.

6. Stemmer library `Sastrawi==1.0.1` is used for stemming words.
7. Also, we remove some stopwords from `nltk.corpus` Indonesian dataset.

Step 6 and 7 is optional, hence I create 3 index:

| Index name                              | Number of unique token (word) | Size   |  Description                  |
|-----------------------------------------|-------------------------------|--------|-------------------------------|
| word_map_not_stemmed_all_word.json      | 92778                         | 31M    | All words is saved as-is, without stemming. I still save into index even if it considered as stopwords. | 
| word_map_stemmed_all_word.json          | 73573                         | 29M    | Save all stemmed words, even if it contains stop words. |
| word_map_stemmed_not_stopword.json      | 73256                         | 18M    | Words are stemmed, and if it is a stopword, then it will not be saved. |


All this index is saved in following JSON format to make it easier to load, read (even by human), and re-use:

```json
{
    "word1": [
        {
            "documentID1": 12
        },
        {
            "documentID2": 10
        }
    ]
}
```

The `12` is the number of occurences of word `word1` in the document with ID `documentID1`.
All arrays is already sorted from high score to lowest before saving.


To build this index from 14343 documents, it took 1 hour 57 minutes 6 seconds on my Apple Macbook M1 Pro CPU 8-Core, GPU 14-Core, RAM 16 GB.

You can see this process in [indexed.ipynb](/indexed.ipynb).


### Retrieving The Documents From The Index

For retrieval, I do write my own logic for boolean logical operator.

1. `OR` operator is pretty simple, if `word` in the query match with the index key, then I push all document IDs. If the same document ID is found for the next query, I pick maximum score to be saved in the list.
2. `AND` operator will only save the document IDs that match of all words in the query.
3. `NOT` operator will look for any document that not contain all word in the query.

I create some testing data for this named [`word_map_test.json`](/word_map_test.json), here's the example:

```json
{
    "a": [
        {"doc1": 3}, 
        {"doc2": 2},
    ], 
    "b": [
        {"doc3": 10},
        {"doc1": 4},
    ], 
    "c": [
        {"doc1": 5},
        {"doc4": 10},
    ],
}
```

| Logical Operator | Query       | Result                                    | Notes |
|------------------|-------------|-------------------------------------------|-------|
| AND              | ['a', 'b']  | [('doc1', 4)]                             | Because only `doc1` that have word `a` and `b`. |
| OR               | ['a', 'b']  | [('doc3', 10), ('doc1', 4), ('doc2', 2)]  | Because either `doc1`, `doc2`, and `doc3` that have word `a` OR `b`. `doc3` is took precedence because it has highest occurence among other documents. |
| NOT              | ['a', 'b']  | [('doc4', 10)]                             | Only `doc4` that not contain both word `a` and `b`. `doc1` contain word `a` and `b`, whereas `doc2` contain word `a`, so these docs is not match with `NOT` query. |


## Analysis

### Computation time

To create index, it tooks: xxx

To retrieve data, the time to query is described in table below:

#### Query: `["pemerintah", "korupsi"]`

| Logical Operator  | Index file                          | Total time |
|-------------------|-------------------------------------|------------|
| AND               | word_map_not_stemmed_all_word.json  |            |
| AND               | word_map_stemmed_all_word.json      |            |
| AND               | word_map_stemmed_not_stopword.json  |            |
|                   |                                     |            |
| OR                | word_map_not_stemmed_all_word.json  |            |
| OR                | word_map_stemmed_all_word.json      |            |
| OR                | word_map_stemmed_not_stopword.json  |            |
|                   |                                     |            |
| NOT               | word_map_not_stemmed_all_word.json  |            |
| NOT               | word_map_stemmed_all_word.json      |            |
| NOT               | word_map_stemmed_not_stopword.json  |            |


#### Query: `["jalan", "rusak"]`

| Logical Operator  | Index file                          | Total time |
|-------------------|-------------------------------------|------------|
| AND               | word_map_not_stemmed_all_word.json  |            |
| AND               | word_map_stemmed_all_word.json      |            |
| AND               | word_map_stemmed_not_stopword.json  |            |
|                   |                                     |            |
| OR                | word_map_not_stemmed_all_word.json  |            |
| OR                | word_map_stemmed_all_word.json      |            |
| OR                | word_map_stemmed_not_stopword.json  |            |
|                   |                                     |            |
| NOT               | word_map_not_stemmed_all_word.json  |            |
| NOT               | word_map_stemmed_all_word.json      |            |
| NOT               | word_map_stemmed_not_stopword.json  |            |


### Are the retrieved documents relevant to the queries?



