Genshin Garbage Collector (G2C)
===============================

Intro
-----

The purpose of this project is to define rules to assess the quality and potential of each artifact allowing for a simpler exclusion of bad artifacts.

### Why use?

Your inventory is always full and you don't have the patience to choose which artifacts to discard.

You want to use wider filters to generate your build, and it's taking hours.

### How it works?

```
+------------------+    +------+    +---------------+    +-----+    +-------------------+
| Inventory Kamera | -> | GOOD | -> | g2c-validator | -> | g2c | -> | Genshin Optimizer |
+------------------+    +------+    +---------------+    +-----+    +-------------------+
                                          /\  ||
                                          ||  \/
                                    +---------------+
                                    |  JSON editor  |
                                    +---------------+
```

- [Inventory Kamera](https://github.com/Andrewthe13th/Inventory_Kamera)
- [GOOD](https://frzyc.github.io/genshin-optimizer/#/doc/)
- [Genshin Optimizer](https://frzyc.github.io/genshin-optimizer/#/)

## Builds

The builds contain two main components: 1) the **filters**; and 2) the **weights** of each sub stats.

Filters are used to limit the artifacts that will be evaluated. Then each artifact receives a score according to the sub stats present.

Undeclared sub stats will be considered as zero.

The weights are normalized when evaluating the artifact score ensuring that every artifact can get points from 0 to 1.

For simplicity's sake, we've abstracted the configuration by creating builds with the relevant artifacts for each character. This ensures that any artifact relevant to a character is preserved.

Each artifact can contain 0 to N scores according to how many builds adhere to it. Additionally, the best score is highlighted in the artifact attributes (in G2C format).

### Output formats

- G2C (Genshin Garbage Collector): used to compute build and scores
- GOOD (Genshin Open Object Description): de facto standard in Genshin community
- Count: amount artifacts returned

### Lock attribute

This attribute indicates the artifacts that must be kept

### List modes

- `keep`: display only artifacts that must be kept (locked)
- `discard`: display only artifacts that must be discarded (unlocked)
- `all`: display all artifacts (locked and unlocked)

### Filters

Filters are an additional layer for excluding artifacts, allowing you to evaluate artifacts by their best score.

Filters contain two main components: 1) the **selectors**; and 2) the **actions**.

Selectors are a comma-separated list of key and value. Below is a list of supported keys and values:

- `set_key`: [GOOD-like values](https://frzyc.github.io/genshin-optimizer/#/doc/)
- `slot_key`: [GOOD-like values](https://frzyc.github.io/genshin-optimizer/#/doc/)
- `main_stat_key`: [GOOD-like values](https://frzyc.github.io/genshin-optimizer/#/doc/)
- `rarity`: `1-5`
- `level`: `0-20`
- `rank`: `0-5` (rank is `floor(level/4)`, i.e. the number of additional rolls an artifact got by leveling)

_*Only artifacts matched by the selector will be filtered._  
_*Any artifact that don't match the selector will be preserved._

It is possible to use a wildcard character to select all artifacts for filtering, i.e. `*:*`.

The actions will effectively filter the list of artifacts and can be as follows:

- threshold (`t`): minimum score threshold to keep artifacts (`float`).
- best score (`b`): select the N best artifacts (`int`).

### Groups

As an alternative to filters, you can use groupings to limit the amount of artifacts kept in each group.

The same filter selector keys apply here (except the wildcard character).

How to run
----------

```
pip install -r requirements.txt
python validator.py -i ~/good-full.json -vvv
python main.py -i '~/good-full.json' -o good > ~/good-filtered.json
```

### Input file

Specify input file in GOOD format.

```
-i/--input-file input_file
```

**Example:**

```
-i './good.json'
```

### Output format

Specify output format (default: g2c).

```
-o/--output-format [g2c|count|good]
```

### List mode

Specify which artifacts to show (default: keep).

```
-k/--keep
-d/--discard
-a/--all
```

### Filters

Filter artifacts according to defined rules.  

```
-f/--filter selector_list=action
  selector_list = selector_key:selector_value[,selector_key:selector_value]
  action = action_type:action_value
```

**Example 1:** keep artifacts above score `0.3`  

```
-f '*:*=t:0.3'
```

**Example 2:** use different score based on artifact rank  

```
-f 'rank:0=t:0.2' -f 'rank:1=t:0.25' -f 'rank:2=t:0.3' -f 'rank:3=t:0.35' -f 'rank:4=t:0.4' -f 'rank:5=t:0.5'
```

**Example 3:** keep artifacts above score `0.4` among `Gladiators Finale` and `Wanderers Troupe` whose rank is between `0` and `2` 

```
-f 'set_key:[GladiatorsFinale,WanderersTroupe],rank:[0,1,2]=t:0.4'
```

**Example 4:** keep the `700` best artifacts  

```
-f '*:*=b:700'
```

**Example 5:** keep the `5` best `plume` from the `Pale Flame` artifact  

```
-f 'set_key:PaleFlame,slot_key:plume=b:5'
```

**Example 6:** keep the `20` best artifacts among `Crimson Witch Of Flames` and `Shimenawas Reminiscence` whose rank is between `0` and `3`  

```
-f 'set_key:[CrimsonWitchOfFlames,ShimenawasReminiscence],rank:[0,1,2,3]=b:20'
```

### Groups

Keeps only N artifacts in each group.

```
-g/--group group_key_list=amount
  group_key_list = set_key,slot_key,main_stat_key,rarity,level,rank
  amount = integer
```

**Examples:**

```
-g 'set_key=25'  # keep the 25 best artifacts each set
-g 'set_key,slot_key=5'  # keep the 5 best artifacts each set and slot
-g 'set_key,slot_key,rank=1'  # keep the best artifact each set, slot and rank
```

### Sort

```
-s/--sort sort_key_list
  sort_key_list = set_key,slot_key,main_stat_key,rarity,level,rank,best_score
```

**Examples:**

```
-s 'best_score'  # sort artifacts from highest to lowest based on best_score attribute
-s 'best_score:desc'  # sort artifacts from highest to lowest based on best_score attribute
-s 'best_score:asc'  # sort artifacts from lowest to highest based on best_score attribute
-s 'level:asc,best_score:desc'  # sort artifacts ascending by level and descending by best_score attribute
```

Rarity 5 Artifact Validator
---------------------------

- Check for null on the first three sub stats
- Check for null on forth sub stat for upgraded artifact
- Check max rarity for artifact sets

How to contribute
-----------------

## Getting started

```shell script
source venv/bin/activate
pip install -r requirements.txt
python main.py -i '~/good.json'
```

## JSON Validator

```shell script
jq . builds/**/*.json > /dev/null 2>&1; echo $?
```

## Artifact formats

- GOOD (Genshin Open Object Description)
- G2C (Genshin Garbage Collector)

## Artifact wrappers

- List: `[artifact]`
- Set/Slot format: `{ set_key: { slot_key: [artifact] } }`
- ID format: `{ id: artifact }`
