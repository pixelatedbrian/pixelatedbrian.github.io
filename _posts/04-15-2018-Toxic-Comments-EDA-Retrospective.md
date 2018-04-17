---
layout: post
title: Toxic Comment Competition
subtitle: Hosted by Kaggle and Google Jigsaw
image: /img/iceberg/iceberg_icon.png
tags: [Retrospective, Classifier, Neural Networks, GRU, EDA, Feature Engineering, Keras, Tensorflow]
---

Recently I competed in a Kaggle / Google Jigsaw Toxic Comment Classification Contest which ended in early March.

<p align="center">
<b>Disclaimer: Clicking on source code linked to in this page will link to offensive words and text</b>
</p>

The first thing that I did was some exploratory data analysis that compared toxic comments with normal comments. I did this to get an idea for some feature engineering to try later. This is because most forms of word vectorization strip the documents of spaces and other punctuation, as well as making all letters lower case.<sup>1</sup> Therefore the concept was to generate some information to concatenate on the model for information that is typically lost before being fed into a model.

While I did do term frequency and most common words per class analysis, given the nature of the subject I'm not including those for now because they're foul, racist, and otherwise terrible.  Also during the initial development of the data pipeline it was discovered that some highly repetitive comments were poisoning some of the statistics. _(Which is why a repetition metric was created, more on this later.)_ 

Anyways, this is being noted up front because the comments analyzed in this EDA had the super repetitive comments cropped to 20 words. So those extreme data didn't have as much of an effect.

**1:** Depending on document processing punctuation could be replaced by reserved words, such as `.` being `<EOS>` but depending on the nuance of the implementation some of this information may still be lost. ie What does one do with `...`, make a `<ELLIPSIS>` reserved character or do `<EOS>` x 3? NLP is hard. : )

## Sentence Structure Features:

### "Words per Comment"

<p align="center">
<img src="/img/words_per_comment.png" width="800" align="middle">
</p>

###### Words per Comment: (redo this graph) _One of the most basic things to look at was to see if there was a difference in length between toxic comments and normal comments. Because it's an exponential distribution a log transformation was also evaluated also._

<p align="center">
<img src="/img/log10_words_per_comment.png" width="800" align="middle">
</p>
###### Log10 Words per Comment: (redo this graph) 

### "Loudness"

Going through and exploring the toxic comments I noticed that a lot of the toxic comments seemed to have more CAPS than usual. Given that is internet 'yelling' it makes sense as toxic messages seemed to be more emotionally charged.

I made a function that counted the proportion of characters that were capitals relative to the total amount of ascii characters. Think `(# A-Z / # a-zA-Z) + ε` Because a log transformation was going to be needed again a small epsilon was added so that the log operation was never performed on a 0 value. Therefore the spike at `~-5.25` are the comments that are all lower case `(0.0 + ε)`

<p align="center">
<img src="/img/log_loudness.png" width="800" align="middle">
</p>

Both have characteristic spikes in _all lower_ and _ALL CAPS_ but the effect is expressed quite a bit more on the toxic comments. This implies that toxic comments are also lower case much more frequently than regular comments.

## Punctuation Based Features:

### "Urgency"
Very similar to loudness, toxic comments seemed to have many more explanation points.

<p align="center">
<img src="/img/urgency_stack.png" width="1200" align="middle">
</p>

###### Distribution of Urgency in Comments: _13% of regular comments had an exclamation mark `!` but 27% of toxic comments had at least one `!`. The plot above has exluded counts of 0 as that makes up the majority of both classes. One can clearly see that the toxic comments are right shifted (more `!`s per comment) relative to normal comments._

### "Questioning"
Do toxic comments have more or less `?` than normal comments, typically?

<p align="center">
<img src="/img/question_stack.png" width="1200" align="middle">
</p>

###### Distribution of Questioning in Comments: _26% of regular comments have a question mark and 26% of toxic comments have question marks.  Overlaying the two plots with the same scale there is a slight right shift of toxic comments but the effect is not pronounced._

### "Dots"
Do toxic comments have more or less `.` or `...` than normal comments, typically? A logical if mundane progression of the statistical punctuation line of thought.

<p align="center">
<img src="/img/dot_stack.png" width="1200" align="middle">
</p>

###### Distribution of Periods in Comments: _13% of normal comments have no `.` but 29% of toxic comments have no `.`. This alone is probably a relatively useful piece of information for a model, which it potentially wouldn't normally have. The toxic comments are again somewhat right shifted so if a toxic comment DOES have `.`s it tends to have more of them. Anecdotally this might be because toxic comments seem to have shorter sentences. So it would be interesting to explore some kind of words per sentence feature in the future, although that would require a bit more cleverness to implement well, given the non-standard prose of the typical toxic comment.