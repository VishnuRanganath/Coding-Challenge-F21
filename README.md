# ACM Research Coding Challenge (Fall 2021)

## Introduction

This document explains my solution to UTD ACM Research's Fall 2021 Coding Challenge. Unfortunately, I lack the technical expertise to implement my solution, so this repository contains only this document and a file with a sample input.

The solution is divided into a number two "steps". Step zero is performed before analyzing anything. Then, step one is performed each time a passage is analyzed.

## Step 0: Establishing a Dictionary

Before a passage can be analyzed for overall sentiment, the words in the passage need to be mapped to scores according to the sentiment that they convey. The score is negative for a negative sentiment and positive for  positive sentiment. Scores range from -1 to 1, both inclusive. Note that the score need not be an integer, as some words are stronger than others.

Creating such a dictionary would be a monumental task due to a variety of different challenges:

- The scale of the English Language
- Homonyms: words that have multiple meanings and therefore multiple sentiment scores

## Step 1: Performing the Sentiment Analysis

A passage's sentiment score can be calculated using a divide-and-conquer approach. A passage can be broken down into paragraphs. Paragraphs contain sentences, sentences contain clauses, and clauses contain words. Therefore, it is possible to use the sentiment scores of individual words to calculate the sentiment scores of a longer passage.

### Computing the Sentiment Score of a Passage

A passage of words is comprised of one or more paragraphs. A paragraph can be defined by a group of sentences with either a newline or nothing before it, and either a newline or nothing after it. Paragraphs often have clear-cut main ideas, which are usually their sole focus. Conversely, it is rare to find two paragraphs with the same main idea, since paragraphs with the same main idea can be merged together. Therefore, computing a passage's sentiment score can be as simple as summing the sentiment scores of its paragraphs. Here is an algorithm that can do so:


```
function computePassageScore(passage pass) {
	score := 0
	for each paragraph par in pass {
		score := score + computeParagraphScore(par)
	}
	return score
}
```

### Computing the Sentiment Score of a Paragraph

A paragraph consists of one or more sentences. Sentences are delineated by punctuation marks - specifically, periods, question marks, and exclamation points. Unlike paragraphs though, sentences do not always have unique main ideas. They can have two or more main ideas. Therefore, when computing the sentiment score of a paragraph, additional information is needed: _context_. The sentiment score of a sentence is partially determined by its neghbors' meanings. Consider the following example:

> The duo killed a total of 10 people.

This sentence would receive a positive score in a paragraph about gaming and a negative score in a paragraph about crime. Creating a list of words that associate with different groups of words is a formidable hurdle.

Here is a high-level algorithm that can score a paragraph:

```
function computeParagraphScore(paragraph par) {
	score := 0
	for each sentence s in par {
		score := score + computeSentenceScore(s) * applyContext(par, s)
	}
	return score
}
```

### Computing the Sentiment Score of a Sentence

A complete sentence consists of one or more clauses. Clauses are groups of words within a sentence, separated by commas, colons, semicolons, hyphens, or ellipses. They are often accompanied by conjunctions (e.g. "and", "but") to help form a relationship between the clauses. Finding the sentiment score of a sentence can be a relatively simple task, since the meanings of different clauses within a sentence are similar. The only additional step required other than taking the sum of the clauses' scores is determining how the clauses' scores should be processed (e.g. addition for "and", subtraction for "but"). Here is an algorithm to compute a sentence's sentiment score:

```
function computeSentenceScore(sentence s) {
	score := 0
	for each clause c in s {
		score := score + computeClauseScore(c) * processConjunction(c)
	}
}
```

### Computing the Sentiment Score of a Clause

A clause, in its most basic form, is just a predicate and a subject. The most basic clause is "I run". Clauses are made up of words and prepositional phrases, but for our purposes prepositional phrases can be treated as strings of words. Finding the sentiment score of a clause can be done by summing the sentiment scores for the words it contains:

```
function computeClauseScore(clause c) {
	score := 0
	for each word in c {
		score := score + word.score
	}
	return score
}
```

## Final Remarks

The algorithm mentioned in this document can assign a score from negative infinity to infinity based on how positive or negative a passage of words is. However, it will not be perfectly accurate as a result of some factors I have missed:

 - Pronouns (e.g. him, it, we) need antecedents to specify what they signify. The algorithm must therefore account for pronouns by replacing them with their respective antecedents prior to analyzing a passage
 - Slang is difficult for the algorithm to analyze because it may contain grammatical mistakes and may use words in vastly different meanings. For example, "no cap" can mean "I'm not lying". Phrases like these can trick the algorithm.
