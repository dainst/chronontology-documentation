# The ChronOntology data model

Wolfgang Schmidle, Nathalie Kallas

Version 0.7 (2017-06-01)

## Introduction

### Goals

The data model reflects the following goals:

* ingest existing data, make semantics explicit
* different levels of completeness and accuracy of the source data
* workflow for enhancing data
* separation between original data and enhancements
* matching between data from different sources


However, we do not attempt to do original research in the project.


### Implementation

ChronOntology records are stored as JSON files. The basic structure of a record is:

```
{
	"resource": {
		
		[1.1 explicitly given information about the period]
		
		[1.2 explicitly given connections to the gazetteer]
		
		"relations": {
			[1.3 explicitly given relations to other periods]
		}
	}

	"derived": {

		[2.1 derived information about the period]

		[2.2 derived connections to the gazetteer]

		"relations": {
			[2.3 derived relations to other periods]
		}
	}
	
	"related": {

		[3. cached records and parts of records from ChronOntology and the gazetteer]
	}

	[4. information about the database record describing the period]
}
```

The ChronOntology back-end keeps the period records as JSON files. In addition, there is an Elasticsearch index. There is no immediate need for putting the data in a "proper" database. The back-end is accessed via an API that provides an abstraction layer over the data storage method.

Via the API one can request single JSONs or all JSONs that conform to an elasticsearch query. Currently, only complete JSONs can be delivered. However, we will implement a featrue for deilivering parts of the JOSNs file.


### Notation for the number of occurrences

The number of occurrences of each element in the JSON is denoted either as a number or with the standard symbols `?`, `*` and `+`:

| abbr. | occurrences |
| --- | --- |
| ? | 0 or 1 |
| * | any number |
| + | at least 1 |

For example "`isSimilarTo`: ?, isSimilarTo list: +" means that `isSimilarTo` may or may not be present but may not be repeated (occurrence: 0 or 1), and contains a non-empty list of IDs (occurrence: at least 1).

TODO: use JSON schema?


## 1. Resource

```javascript
{
	"resource": {
```

`resource`: 1

##### comments:
`resource` contains explicitly given information about the period.
All information from the import table / data editor goes here. 

### 1.1 information about the period itself

Fields in parentheses ( ) are filled in by the system and can not be set by the user.

#### (id)

```javascript
		"id": "0Bq31JfToIUY",
```

`id`: 1

##### comments:

The ID is only visible to the user as part of a URI such as 
http://chronontology.dainst.org/period/KTwRym1w8abB

Importing data into the system with:

* http POST: The system creates a new random ID.
* http PUT: The system uses the specified ID. An existing record with this ID will be overwritten. (The record will still exist in the file system, but there is currently no API for retrieving older versions.)

#### (type)

```javascript
		"type": "period",
```

`type`: 1

##### comments:

This technical information is not directly visible to the user. Not to be confused with `types`, see below. 

The value "period" is hardcoded in ChronOntology, so the import script needs to specify exactly this value. It corresponds to the `/period/`in the URI, as in 
http://chronontology.dainst.org/period/KTwRym1w8abB

One might add other types such as "event". Another possible type would be "collection", see "lists / isListedIn" below. However, since "period" means something like "this is a dataset in ChronOntology", periods versus events should better be marked by a type value within the data model. 

TODO: Check if this is correct. One might want to mark the different types in the URI as well. (What are the consequences in the back-end? Different mapping file?)

#### externalId

```javascript
		"externalId": "abc:12345",
```

##### comments:
If the period already has an ID in the original (electronic) data source.

TODO: use URIs rather than IDs where possible? E.g.  http://vocab.getty.edu/aat/300019571 rather than aat:300019571 ?

#### names

```javascript
		"names": {
			"de": [ "..." ],
			"en": [ "...", "..." ]
		},
```

`names`: 1, language block: +, content list for each given language code must be non-empty

##### examples:

We so far there are English (en), German (de) and Arabic (ar) period names in the data.

```
Prähistorie@de
Urgeschichte@de
Vorgeschichte@de
Prehistory@en

Early Copper (MA I-II)@en

Period 1@en

Br. ancien I@fr
Bronze ancien I@fr

Taifa-Reiche@de
```

(The arabic name for Taifa-Reiche, "طائفة‎@ar", is too difficult for the Markdown Editor and the fixed-width font it uses.)

##### comments:

The period page displays the preferred name in the preferred language. The first term in each language is the preferred term for this language. The preferred language is 

* the language of the user's browser preferences, or 
* English, or 
* German, or
* the first language in `names` (i.e. language codes, alphabetically sorted)

Thus, the order of languages in `names` is relevant only if neither the user's preferred language nor English or German exist. 

The frontend and backend accept any language code. However, the language codes are checked by the import script. The list of allowed languages is taken from the iDAI.gazetteer, which consists of roughly 20 language codes. Currently, the data contains names in `de`, `en` and `ar`. 

TODO: ISO 639-3 language codes

#### types

```javascript
		"types": [ "Geological Epoch" ],
```

`types`: 1, types list: * (or "+"? most often 1)

##### examples:

| type | description | examples |
| --- | --- | --- |
| Geological | A system of chronological measurement that relates stratigraphy to time. These periods form elements of a hierarchy of divisions into which geologists have split the Earth's history. There are generally divided into 6 levels: Supereon, Eon, Era, Period, Epoch, Age | Phanerozoic, Paleozoic, Holocene | 
| Political | Defined based on historical events that have been recorded and can be more or less precisely dated. This could be a reign of a certain king or ruler, or the period when a certain kingdom, empire, independent entry existed. Could also eventually be used for war periods. | the rule of the roman Augustus, the rule of the Ottoman Mehmed the Conqueror, the roman empire, the Nubian kingdom; also valid for WW II | 
| Cultural | Are related to the existence (appearance/disappearance) of a specific culture or cultural group, regardless of the contemporary political, technological or material developments. | Mudéjar, Philistine, Pheonician, Italian | 
| Material culture | Defined based on art movement or the production of artifact according to a specific style. This is valid for all art styles and for archaeological periods defined based on changes in material culture and technology. Such as, pottery, stone tools, textile, paintings, sculpture, architecture | Art Deco, Italian Renaissance, Neoclassicism, Realism, Russian avant-garde; Paleolithic, Azilien, Bronze Age, Stone Age, Geometrisch | 
| Technological | Based on concrete technological development that significantly altered | Industrial Age, Space Age, Information Age, Atomic Age, Nuclear Age | 
| Abbreviated Century/Millenium/etc. | A reference to specific century without link to defined events. | 2nd Millemium BC, 17th Century AD | 
| Chronological subdivision | Subdivision of any type of periods to smaller entities, not necessarily linked with specific developments related to the defining criteria of the period. | Early modern period, 1rst half of 17th c, mid 20th century | 
| alle Bedeutungen | When not specified in source | Valid for all | 

TODO: Do I agree with the descriptions and examples? 

* war = political? why not own type? Are there examples in the data?

TODO: type hierarchy

TODO: Typen from the Forth paper


##### comments:

If the type is not explicitly given in the original source and can (or has not) not be inferred, the default type is "all meanings".

TODO: rename: clashes with `type`, and means the same as "senses". In addition, there is the type "all meanings". Find a  common term.

Periods need to be defined, but types need to be defined, too. Problem of different definitions of the "same" type, which makes matching difficult. The maximal position is that virtually no two definitions / types are the same, so there would be no records containing different datings for the same thing. However, at least the reign of kings is normally well-defined enough to assume that differen researchers are talking about the same thing.


#### provenance


```javascript
		"provenance": "chronontology",
```

`provenance`: 1

##### comments:

If the provenance is missing or left empty, the import script adds the default provenance "chronontology".

#### definition

```javascript
		"definition": "...",
```

`definition`: ?

##### comments:

TODO: also split up languages as in `names`? (see `description`)

#### description

```javascript
		"description": "...",
```

`description`: ?

##### comments:

TODO: move `description` text in different languages and source to separate subfields, as with `names`

#### tags

```javascript
		"tags": [
			"..."
		],
```

`tags`: ?, tags list: +

#### note fields

```javascript
		"note": "c-2",
		"note2": "c-2",
		"note3": "c-2",
```

`note`: ?

is visible in the ChronOntology frontend

TODO: also split up languages as in `names`? (see `description`)

`note2`: ?

contains internal notes that should be visible only to registered users

`note3`: ?

may or may not be superfluous

#### timespan fields

```javascript
		"ongoing": true,
		"hasTimespan": [
			{
				"sourceOriginal": "Wikipedia deutsch",
				"sourceURL": "http://...",
				"timeOriginal": "Serie: Oberkreide (100,5–66 Ma)",
				"calendar": "gregorian",
				"begin": {
					"at": "-100500000",
					"atPrecision": "ca"
				},
				"end": {
					"at": "-66000000",
					"atPrecision": "ca"
				}
			}
		],
```

`ongoing`: ? (if missing, the default is "false")
`hasTimespan`: ?, time block: +
`sourceOriginal`, `sourceURL`, `timeOriginal`, `calendar`: ?
`begin`, `end`: ?
`notBefore`, `notAfter`: ?
`at`: ?, `atPrecision`: ? (only makes sense if `at` is specified)

TODO: rework time representation

TODO: examples for `notBefore` and `notAfter`

TODO: Spacetime volumes


### 1.2 connections to the gazetteer

Currently these fields are on the same level as `id`, `names`, `types` etc., unlike the relations to other ChronOntology records. However, this might change. 

Currently we only have connections to the iDAI.gazetteer, which in turn links to other gazetteers. An enricher script will then add the palce name in `related` (see below). 

However, one can also include URLs of other gazetteers directly. 

TODO: The enricher script will then ask a dedicated "spatial service" to extract the place name. 

#### spatiallyPartOfRegion

```javascript
		"spatiallyPartOfRegion": [
			"http://gazetteer.dainst.org/place/2359913"
		]
```

`spatiallyPartOfRegion`: ?, spatiallyPartOfRegion list: + (more than 1 if the period can be best described as being part of more than one region given in the gazetteer)

#### hasCoreArea

```javascript
		"hasCoreArea": [
			"http://gazetteer.dainst.org/place/2359913"
		]
```

`hasCoreArea`: ?, hasCoreArea list: + 

#### isNamedAfter

```javascript
		"isNamedAfter": [
			"http://gazetteer.dainst.org/place/2359913"
		]
```

`isNamedAfter`: ?, isNamedAfter list: + (list with more than 1 value will be very rare)

### 1.3 explicitly stated relations to other periods

```javascript
		"relations": {
```

`relations`: ? (normally 1)

##### comments:

TODO: Does the grouping "parent / senses / siblings / matching / Allen" make sense? (Has replaced "relations that hold by definition".) Argument for this grouping: will be displayed in similar ways in the Frontend.

#### 1.3.1 parent relations

A bracket for all kinds of parent/child relations. 

##### hasPart / isPartOf

```javascript
			"hasPart": [
				"OSNu4KDy6piw"
			],
			"isPartOf": [
				"0ORH5IjCY2oU"
			],
```

`hasPart`: ?, hasPart list: + 
`isPartOf`: ?, isPartOf list: + (most often 1)

##### comments:

This means that a period is by definition part of another period.

##### fallsWithin / contains

```javascript
			"follows": [
				"0ORH5IjCY2oU"
			],
			"isFollowedBy": [
				"OSNu4KDy6piw"
			],
```

`fallsWithin`: ?, fallsWithin list: + (most often 1)
`contains`: ?, contains list: + 

##### comments:

This corresponds to the Cidoc CRM property "P10 falls within / contains" between STVs.

##### lists / isListedIn

```javascript
			"lists": [
				"0ORH5IjCY2oU"
			],
			"isListedIn": [
				"OSNu4KDy6piw"
			],
```

`lists`: ?, "lists" list: + 
`isListedIn`: ?, isListedIn list: + (most often 1)

##### comments:

For wrappers around collections of periods etc., especially if these wrappers are present in already exisitng systems such as the Getty AAT. The wrapper itself is not a period. It corresponds to a SKOS collection.

#### 1.3.2 senses relations 

##### hasSense / isSenseOf

```javascript
			"hasSense": [
				"0ORH5IjCY2oU"
			],
			"isSenseOf": [
				"OSNu4KDy6piw"
			],
```

`hasSense`: ?, hasSense list: + 
`isSenseOf`: ?, isSenseOf list: + (most often 1)

#### 1.3.3 sibling relations 

##### follows / isFollowedBy

```javascript
			"follows": [
				"0ORH5IjCY2oU"
			],
			"isFollowedBy": [
				"OSNu4KDy6piw"
			],
```

`follows`: ?, follows list: + (most often 1)
`isFollowedBy`: ?, isFollowedBy list: + (most often 1)

#### 1.3.4 matching relations

Matching will be a focucs in the last part of the project. It is very likely that the data model for matching statements will change.

##### sameAs

```javascript
			"sameAs": [
				"75u4mI70rXt2"
			],
```

`sameAs`: ?, sameAs list: + 

##### comments:

TODO: The semantics of `sameAs` is not the strict sameAs from SKOS. Clarify and rename.

##### isSimilarTo

```javascript
			"isSimilarTo": [
				"75u4mI70rXt2"
			],
```

`isSimilarTo`: ?, isSimilarTo list: + 

##### comments:

The semantics of isSimilarTo is not completely fixed yet. An example is 

```
A0861 Late Mycenaean, all meanings, provenance Arachne
    isSimilarTo A0829 LH III A, all meanings, provenance Arachne
    isSimilarTo A0836 LH III B, all meanings, provenance Arachne
```

##### isEqualOrFinerThan

We thought of a verb `isEqualOrFinerThan` for the situation that we cannot establish a precise correspondence of types, for example if we are not sure if a period is meant as "all meanings" or with a specific type. This is particularly useful for ingesting larger amounts of data with limited preprocessing capacity, while at the same time making vague but correct ststements.

However, there seems to be no example in the data (yet).

#### 1.3.5 non-causal temporal relations

```javascript
			"includes": [ "0ORH5IjCY2oU" ],
			"occursDuring": [ "0ORH5IjCY2oU" ],
			"meetsInTimeWith": [ "0ORH5IjCY2oU" ],
			"isMetInTimeBy": [ "0ORH5IjCY2oU" ],
			"occursBefore": [ "0ORH5IjCY2oU" ],
			"occursAfter": [ "0ORH5IjCY2oU" ],
			"isEqualInTimeTo": [ "0ORH5IjCY2oU" ],
			"starts": [ "0ORH5IjCY2oU" ],
			"isStartedBy": [ "0ORH5IjCY2oU" ],
			"finishes": [ "0ORH5IjCY2oU" ],
			"isFinishedBy": [ "0ORH5IjCY2oU" ],
			"overlapsInTimeWith": [ "0ORH5IjCY2oU" ],
			"isOverlappedInTimeBy": [ "0ORH5IjCY2oU" ],
			"startsAtTheEndOf": [ "0ORH5IjCY2oU" ],
			"endsAtTheStartOf": [ "0ORH5IjCY2oU" ],
```

All Allen relations: ?, the respective lists: +

##### comments:

Note that these relations refer to time intervals rather than STVs. The standard interpretation would be to take the projections to the timeline and compare the projections.

Explicitly stated Allen relations between time intervals will be quite rare. The main place for Allen relations in is the `derived` part.

The Allen relations may be complemented and/or replaced by "temporal primitives":

```javascript
			"startsAtTheEndOf": [ "0ORH5IjCY2oU" ],
			"endsAtTheStartOf": [ "0ORH5IjCY2oU" ],
			...
```

## 2. Derived

```javascript
	"derived": {
```		

All information that is not explicitly given in the record, but is derived from this (and possibly other) information. This part can always be deleted and re-created.

### 2.1 derived information about the period

Not implemented yet. No examples in the data.

### 2.2 derived connections to the gazetteer

A child inherits the bounding polygon of the parent. the other direction is more tricky.

Not implemented yet. 

### 2.3 derived relations to other periods

```javascript
		"relations": {
			... 
		}
	},
```

In priciple, all relations in `resource` can also occur in `derived`. 

The reasoning takes place in the import script for now. However, it will be replaced by separate scripts that will be triggered by the back-end whenever there is a change in the data.

Currently, three reasoning steps are implemented. 

1. causal (STV) => de facto (time interval)
  * isPartOf => occursDuring, hasPart => includes
  * isFollowedBy => meetsInTimeWith, follows => isMetInTimeBy

2. causal (STV) => de facto (STV)
  * isPartOf => fallsWithin, hasPart => contains

3. causal (STV) => de facto (temporal primitives)
  * isFollowedBy => endsAtTheStartOf, follows => startsAtTheEndOf
  
Some of these reasonings are only experimental, in particular "causal (STV) => de facto (fuzzy interval)". Also, for example "follows => isMetInTimeBy" may not always be strictly true with the standard interpretation of Allen relations between STVs given above.

All these reasoning steps are within a record, but of course there are many other possible reasonings, for example inheriting geographic information between parent and child.

For the numbers of occurrences see `resource`. 

TODO: Temporal primitives

## 3. Related

`related` contains cached records or parts of records. Currently we only cache the names of gazetteer entries. For example:

```
  "related": {
    "http://gazetteer.dainst.org/place/2181146": {
      "prefName": {
        "title": "Nótio Aigaío (Periféreia)",
        "language": "ell",
        "transliterated": true
      }
    }
  }
```

Caching gazetteer records saves additional on-the-fly queries to the gazetteer. However, there is a tradeoff in that it makes the ChronOntology records bigger. For example, for performance reasons we can't store complete gazetteer records including polygons. 


## 4. information about the database record

Information that is not in `resource`, `derived` or `related` refers to the database record about the period rather than the period itself.

An example:

```javascript
  "dataset": "none",
  "version": 3,
  "created": {
    "user": "admin",
    "date": "2017-05-29T10:27:28.099+02:00"
  },
  "modified": [
    {
      "user": "admin",
      "date": "2017-05-29T10:32:15.057+02:00"
    },
    {
      "user": "admin",
      "date": "2017-05-30T13:52:12.907+02:00"
    },
}
```

`dataset`, `version`: 1
`created`: 1, `modified`: ?, modified block: +, `user`, `date`: 1


## Examples

TODO: examples without Wikipedia material

### Example 1: Roman, type "political", provenance "chronontology"

http://chronontology.dainst.org/period/KTwRym1w8abB

### Example 2: Augustan

### Example 3: Hadrianic

### Example 4: A Geological Epoch


## Upcoming changes in the data model

The data model is not finished yet.

Bias towards the already ingested data, mainly from electronic sources.

Things that need to be reworked are for example:

* The semantics of the verbs are not entirely fixed yet. In particular, there are no Cidoc CRM mappings yet.

* matching relations

* a unified data model for places and periods


### Implementing changes 

Changes in the data model need to be implemented in 

* the ChronOntology Frontend, including the data editor (https://github.com/dainst/chronontology-frontend)
* The Backend (https://github.com/dainst/jeremy) is mostly data model-agnostic. However, one must update the mapping file (https://github.com/dainst/jeremy/blob/master/src/main/resources/mapping.json)
* the ChronOntology import scripts (https://github.com/dainst/chronontology-data, private repository)
* transl8 (the translation service for a multilingual unser interface, see e.g. https://arachne.dainst.org/transl8/translation/json?application=chronontology_frontend&lang=de&callback=angular.callbacks._0): The frontend expects some naming conventions in transl8. For example, "isFollowedBy" becomes `relation_isFollowedBy`.

## Using ChronOntology

common use cases


## TODO

* Examples for each verb (from the data)
