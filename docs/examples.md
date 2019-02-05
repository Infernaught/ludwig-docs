Examples
========

In this section you will find several examples of how to build models with Ludwig for a variety of tasks.
For each task we show an example dataset and a sample model definition that can be used to train a model from that data.

Basic Kaggle-completition-like example (titanic / house pricing)
----------------------------------------------------------------

Text Classification
-------------------

| text                                                                                             | class       |
|--------------------------------------------------------------------------------------------------|-------------|
| Toronto  Feb 26 - Standard Trustco said it expects earnings in 1987 to increase at least 15..   | earnings    |
| New York  Feb 26 - American Express Co remained silent on market rumors..                       | acquisition |
| BANGKOK  March 25 - Vietnam will resettle 300000 people on state farms known as new economic.. | coffee      |

```
ludwig experiment \
  --data_csv reuters-allcats.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: text
        type: text
        encoder: parallel_cnn
        level: word
output_features:
    -
        name: class
        type: category
```

Named Entity Recognition Tagging
--------------------------------

| utterance                                         | tag                                            |
|---------------------------------------------------|------------------------------------------------|
| John Smith was born in New York on July 21st 1982 | Person Person O O O City City O Date Date Date |
| Jane Smith was born in Boston on May 1st 1973     | Person Person O O O City City O Date Date Date |
| My friend Carlos was born in San Jose             | O O Person O O O City City                     |

```
ludwig experiment \
  --data_csv sequence_tags.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: utterance
        type: sequence
        encoder: rnn
        cell_type: lstm
        reduce_output: null
output_features:
    -
        name: tag
        type: sequence
        decoder: tagger
```

Natural Language Understanding
------------------------------

| utterance                      | intent                            | slots       |
|--------------------------------|-----------------------------------|-------------|
| I want a pizza                 | O O O B-Food_type                 | order_food  |
| Book a flight to Boston        | O O O O B-City                    | book_flight |
| Book a flight at 7pm to London | O O O O B-Departure_time O B-City | book_flight |

```
ludwig experiment \
  --data_csv reuters-allcats.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: utterance
        type: sequence
        encoder: rnn
        cell_type: lstm
        bidirectional: true
        num_layers: 2
        reduce_output: None
output_features:
    -
        name: intent
        type: category
        reduce_input: sum
        num_fc_layers: 1
        fc_size: 64
    -
        name: slots
        type: sequence
        decoder: tagger
```

Language Modeling
-----------------

Machine Translation
-------------------

| english                   | italian                   |
|---------------------------|---------------------------|
| Hello! How are you doing? | Ciao, come stai?          |
| I got promoted today      | Oggi sono stato promosso! |
| Not doing well today      | Oggi non mi sento bene    |

```
ludwig experiment \
  --data_csv translation.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: english
        type: sequence
        encoder: rnn
        cell_type: lstm
output_features:
    -
        name: italian
        type: sequence
        decoder: generator
        cell_type: lstm
        attention: bahdanau
```

Chit-Chat Dialogue Modeling through Sequence2Sequence
-----------------------------------------------------

| user1                     | user2                                      |
|---------------------------|--------------------------------------------|
| Hello! How are you doing? | Doing well, thanks!                        |
| I got promoted today      | Congratulations!                           |
| Not doing well today      | I’m sorry, can I do something to help you? |

```
ludwig experiment \
  --data_csv chitchat.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: user1
        type: sequence
        encoder: rnn
        cell_type: lstm
output_features:
    -
        name: user2
        type: sequence
        decoder: generator
        cell_type: lstm
        attention: bahdanau
```

Sentiment Analysis
------------------

| review                          | sentiment |
|---------------------------------|-----------|
| The movie was fantastic!        | positive  |
| Great acting and cinematography | positive  |
| The acting was terrible!        | negative  |

```
ludwig experiment \
  --data_csv reuters-allcats.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: review
        type: text
        encoder: parallel_cnn
        level: word
output_features:
    -
        name: sentiment
        type: category
```

Image Classification
--------------------

| image_path                | class |
|---------------------------|-------|
| imagenet/image_000001.jpg | car   |
| imagenet/image_000002.jpg | dog   |
| imagenet/image_000003.jpg | boat  |

```
ludwig experiment \
  --data_csv reuters-allcats.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: image_path
        type: image
        encoder: stacked_cnn
output_features:
    -
        name: class
        type: category
```

Image Captioning
----------------

| image_path                | caption                   |
|---------------------------|---------------------------|
| imagenet/image_000001.jpg | car driving on the street |
| imagenet/image_000002.jpg | dog barking at a cat      |
| imagenet/image_000003.jpg | boat sailing in the ocean |

```
ludwig experiment \
--data_csv reuters-allcats.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: image_path
        type: image
        encoder: stacked_cnn
output_features:
    -
        name: caption
        type: sequence
        decoder: generator
        cell_type: lstm
```

Siamese Netowrk
---------------

Visual Question Answering
-------------------------

------------------------------------
| image_path              |   question                                | answer |
|-------------------------|-------------------------------------------|--------|
| imdata/image_000001.jpg | Is there snow on the mountains?           | yes    |
| imdata/image_000002.jpg | What color are the wheels                 | blue   |
| imdata/image_000003.jpg | What kind of utensil is in the glass bowl | knife  |

```yaml
input_features:
    -
        name: image_path
        type: image
        encoder: stacked_cnn
    -
        name: question
        type: text
        encoder: parallel_cnn
        level: word
output_features:
    -
        name: answer
        type: sequence
        decoder: generator
        cell_type: lstm
```


Time series forecasting
-------------------------

While direct timeseries prediction is a work in progress Ludwig can ingest timeseries input feature data and make numerical predictions. Below is an example of a model trained to forecast timeseries at five different horizons.

----------------------
| timeseries_data       |   y1  |   y2  |   y3  |   y4  |   y5  |
|-----------------------|-------|-------|-------|-------|-------|
| 15.07 14.89 14.45 ... | 16.92 | 16.67 | 16.48 | 17.00 | 17.02 |
| 14.89 14.45 14.30 ... | 16.67 | 16.48 | 17.00 | 17.02 | 16.48 |
| 14.45 14.3 14.94 ...  | 16.48 | 17.00 | 17.02 | 16.48 | 15.82 |

```
ludwig experiment \
--data_csv timeseries_data.csv \
  --model_definition model_definition.yaml
```

With `model_definition.yaml`:

```yaml
input_features:
    -
        name: timeseries_data
        type: timeseries
output_features:
    -
        name: y1
        type: numerical
    -
        name: y2
        type: numerical
    -
        name: y3
        type: numerical
    -
        name: y4
        type: numerical
    -
        name: y5
        type: numerical
```

Movie Rating prediction
----------------------

| year | duration  | nominations |  categories        | rating
|------|-----------|-------------|--------------------|-------|
| 1921 |   3240    |     0       | comedy drama       |  8.4  |
| 1925 |   5700    |     1       | adventure comedy   |  8.3  |
| 1927 |   9180    |     4       | drama comedy scifi |  8.4  |

```yaml
input_features:
    -
        name: year
        type: numerical
    -
        name: duration
        type: numerical
    -
        name: nominations
        type: numerical
    -
        name: categories
        type: set
        
output_features:
    -
        name: rating
        type: numerical
```

Example that uses Sets/Bags
---------------------------

| image_path                | tags  |
|---------------------------|-------|
| imagenet/image_000001.jpg | car, man   |
| imagenet/image_000002.jpg | happy, dog, tie   |
| imagenet/image_000003.jpg | boat, water  |

```yaml
input_features:
    -
        name: image_path
        type: image
        encoder: stacked_cnn
output_features:
    -
        name: tags
        type: set
```

Example of Multi-Task Learning
------------------------------