---
title: DependencyMatcher
teaser: Match on dependency paths using semregex syntax
tag: class
source: spacy/matcher/dependencymatcher.pyx
---



## DependencyMatcher.\_\_init\_\_ {#init tag="method"}

Create the DependencyMatcher.

> #### Example
>
> ```python
> from spacy.matcher import DependencyMatcher
> matcher = DependencyMatcher(nlp.vocab)
> ```

| Name                                    | Type      | Description                                                                                 |
| --------------------------------------- | --------- | ------------------------------------------------------------------------------------------- |
| `vocab`                                 | `Vocab`   | The vocabulary object, which must be shared with the documents the matcher will operate on. |
| **RETURNS**                             | `DependencyMatcher` | The newly constructed object.                                                               |

## DependencyMatcher.\_\_call\_\_ {#call tag="method"}

Match a dependency path of the supplied pattern in the `Doc`.

> #### Example
>
> ```python
> import spacy 
> from spacy.matcher import DependencyMatcher
> from spacy.pipeline import merge_entities
>
> nlp = spacy.load('en_core_web_sm') 
> nlp.add_pipe(merge_entities)
>
> pattern = [{'SPEC': {'NODE_NAME': 'founded'}, 'PATTERN': {'ORTH': 'founded'}},
>            {'SPEC': {'NODE_NAME': 'START_ENTITY', 'NBOR_RELOP': '>', 'NBOR_NAME': 'founded'},
>             'PATTERN': {'DEP': 'nsubj', 'ENT_TYPE': {'NOT_IN': ['']}}},
>            {'SPEC': {'NODE_NAME': 'END_ENTITY', 'NBOR_RELOP': '>', 'NBOR_NAME': 'founded'},
>             'PATTERN': {'DEP': 'dobj', 'ENT_TYPE': {'NOT_IN': ['']}}}]
>
> matcher = DependencyMatcher(nlp.vocab)
> matcher.add("pattern1", None, pattern)
>
> doc1 = nlp("Bill Gates founded Microsoft.")
> doc2 = nlp("Bill Gates, the Seattle Seahawks owner, founded Microsoft.")
>
> match = matcher(doc1)[0]
> subtree = match[1][0]
> print("Subtree:", subtree)  # prints -> Subtree: [1, 0, 2]
>
> match = matcher(doc2)[0]
> subtree = match[1][0]
> print("Subtree:", subtree)  # prints -> Subtree: [6, 0, 7]
> ```

| Name        | Type  | Description                                                                                                                                                              |
| ----------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `doc`       | `Doc` | The document to match over.                                                                                                                                              |
| **RETURNS** | list  | A list of `(match_id, start, end)` tuples, describing the matches. A match tuple describes a span `doc[start:end`]. The `match_id` is the ID of the added match pattern. |


## DependencyMatcher.\_\_len\_\_ {#len tag="method" new="2"}

Get the number of rules, which are edges, added to the dependency
        tree matcher.

> #### Example
>
> ```python
> matcher = DependencyMatcher(nlp.vocab)
> assert len(matcher) == 0
> matcher.add("Rule", None, [{"ORTH": "test"}])
> assert len(matcher) == 1
> ```

| Name        | Type | Description          |
| ----------- | ---- | -------------------- |
| **RETURNS** | int  | The number of rules. |

## DependencyMatcher.\_\_contains\_\_ {#contains tag="method" new="2"}

Check whether the matcher contains rules for a match ID.

> #### Example
>
> ```python
> matcher = DependencyMatcher(nlp.vocab)
> assert 'Rule' not in matcher
> matcher.add('Rule', None, [{'ORTH': 'test'}])
> assert 'Rule' in matcher
> ```

| Name        | Type    | Description                                           |
| ----------- | ------- | ----------------------------------------------------- |
| `key`       | unicode | The match ID.                                         |
| **RETURNS** | bool    | Whether the matcher contains rules for this match ID. |

## DependencyMatcher.add {#add tag="method" new="2"}

Add a rule to the matcher, consisting of an ID key, one or more patterns, and a
callback function to act on the matches. The callback function will receive the
arguments `matcher`, `doc`, `i` and `matches`. If a pattern already exists for
the given ID, the patterns will be extended. An `on_match` callback will be
overwritten.

> #### Example
>
> ```python
> def on_match(matcher, doc, id, matches):
>      print('Matched!', matches)
> 
> matcher = DependencyMatcher(nlp.vocab)
> matcher.add("HelloWorld", on_match, [{"LOWER": "hello"}, {"LOWER": "world"}])
> matcher.add("GoogleMaps", on_match, [{"ORTH": "Google"}, {"ORTH": "Maps"}])
> doc = nlp("HELLO WORLD on Google Maps.")
> matches = matcher(doc) 
> ```

| Name        | Type               | Description                                                                                   |
| ----------- | ------------------ | --------------------------------------------------------------------------------------------- |
| `match_id`  | unicode            | An ID for the thing you're matching.                                                          |
| `on_match`  | callable or `None` | Callback function to act on matches. Takes the arguments `matcher`, `doc`, `i` and `matches`. |
| `*patterns` | list               | Match pattern. A pattern consists of a list of dicts, where each dict describes a token.      |

<Infobox title="Changed in v2.2.2" variant="warning">

As of spaCy 2.2.2, `DependencyMatcher.add` also supports the new API, which will become
the default in the future. The patterns are now the second argument and a list
(instead of a variable number of arguments). The `on_match` callback becomes an
optional keyword argument.

```diff
patterns = [[{"TEXT": "Google"}, {"TEXT": "Now"}], [{"TEXT": "GoogleNow"}]]
- matcher.add("GoogleNow", None, *patterns)
+ matcher.add("GoogleNow", patterns)
- matcher.add("GoogleNow", on_match, *patterns)
+ matcher.add("GoogleNow", patterns, on_match=on_match)
```

</Infobox>


## DependencyMatcher.get {#get tag="method" new="2"}

Retrieve the pattern stored for a key. Returns the rule as an
`(on_match, patterns)` tuple containing the callback and available patterns.

> #### Example
>
> ```python
> matcher.add("Rule", None, [{"ORTH": "test"}])
> on_match, patterns = matcher.get("Rule")
> ```

| Name        | Type    | Description                                   |
| ----------- | ------- | --------------------------------------------- |
| `key`       | unicode | The ID of the match rule.                     |
| **RETURNS** | tuple   | The rule, as an `(on_match, patterns)` tuple. |
