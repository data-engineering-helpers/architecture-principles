Data platform - Architecture principles
=======================================

# Overview
[This project](https://github.com/data-engineering-helpers/architecture-principles)
intends to collaborate on specifying architecture principles and diagrams
for a typical data platform with the so-called Modern Data Stack (MDS).

Even though the members of the GitHub organization may be employed by
some companies, they speak on their personal behalf and do not represent
these companies.

## References
* [Material for the Data platform - Architecture principles](material/)
* Specifications/principles for a
  [data engineering pipeline deployment tool](https://github.com/data-engineering-helpers/data-pipeline-deployment)

# Diagrams

## Data lake - ins and outs
* [Data engineering Excalidraw diagram online - Data platform principles for data lake ins and outs](https://excalidraw.com/#json=mv7jSkpTewcQb_S4raJ5G,S6aAoK8gA3VroJ5ai8Kb6w)

* [Excalidraw source on GitHub - Data platform principles for data lake ins and outs](https://github.com/data-engineering-helpers/architecture-principles/blob/main/diagrams/Data%20Platform%20-%20Principles%20-%20Data%20Lake%20In%20and%20Out%20-%20latest.excalidraw)

![Data platform principles for data lake ins and outs](https://github.com/data-engineering-helpers/architecture-principles/blob/main/diagrams/snapshots/Data%20Platform%20-%20Principles%20-%20Data%20Lake%20In%20and%20Out%20-%202023-04%20-%20v2.0.png)

## Data engineering
* [Data engineering Excalidraw diagram online - Data platform principles for Data Engineering](https://excalidraw.com/#json=UPsnozgpMAxRaz3feC23y,n478x5MVcgCz1XTZ7h9qHw)

* [Excalidraw source on GitHub - Data platform principles for Data Engineering](https://github.com/data-engineering-helpers/architecture-principles/blob/main/diagrams/Data%20Platform%20-%20Principles%20-%20Data%20Engineering%20-%20latest.excalidraw)

![Data Platform - Principles - Data Engineering](https://github.com/data-engineering-helpers/architecture-principles/blob/main/diagrams/snapshots/Data%20Platform%20-%20Principles%20-%20Data%20Engineering%20-%202023-04%20-%20v2.1.png)

# Principles

## Production vs non-production
As a summary, production and non-production environments should be
as separated as possible, if possible separated by kind of a "Chinese wall":
* Non-production environments should not have access, by design,
  to production resources, including production data
* The only allowed tasks are the publication of non sensitive data,
  by production environment (_e.g._, Spark processes) to non-production
  storage (S3 buckets)
* As data scientists, analysts and engineers have to be able to work
  on realistic data sets, the above principles mean that teams must invest
  on how to create non sensitive data from production data. In order to do so,
  several processes are possible (_e.g._, anonymisation, obfuscation, aggregation,
  data generation, simulation). Some specialized companies, such as
  [Statice+Anonos](https://www.statice.ai/), help in generating
  non sensitive realistic data sets.

## Persistency of data files
Once the data files are written to S3, they must never be overwritten.
The data files are stored in S3 in a persistent way, versioned (_e.g._,
with [Delta](https://delta.io/)), and must be kept on cloud object storage
(_e.g._, AWS S3, Azure ADLS, Google GCS) as long as legally and
technically possible.

That principle is the same as the one in the
[Change Data Capture (CDC)](https://en.wikipedia.org/wiki/Change_data_capture)
mechanism:
* Regularly, snapshots are taken out of a given data set
* Snapshots take the shape of Parquet/Delta data files. Functionally, a snapshot
  is similar to a picture taken with a photo camera: it corresponds to the latest
  state of the data set, consistent and instantaneous (there is no history in
  a snapshot)
* Snapshots must be versioned. Usually, it is enough to add the time-stamp
  of when the snapshot was taken to the file-path/URI of the snapshot data files
* The succession of the snapshots correspond to the succession of the versions
  of the data set
* Snapshots data files must be persistent: they must never be overwritten
* The history may be rebuilt from the succession of the snapshots

The [Delta format](https://delta.io/) applies that principle of keeping persistent
snapshots/versions of a given data set, while abstracting away the need to version
and to not overwrite data sets. With Delta, one can store and "overwrite" data sets,
while in practice the data files are actually versioned snapshot data files and
the log of transactions is kept along those snapshots/versions so as to allow
rebuilding of the history.

## Format of data files
The format of the structured data files must be Delta wherever possible,
and Parquet only when Delta is not possible. No other data format is allowed
on the data lake for structured data.

## Data processing, from files to files
Any data processing task:
1. Makes use of software artifacts (_e.g._, Python wheels,
   Scala JARs, dbt SQL artifacts)
2. Takes, as input, data files, which are, as mentioned above,
   persistent and versioned, _i.e._, which will never be overwritten
   and which their version allows to uniquely identify
3. Generates, as output, data files, which have to be, as mentioned above,
   persistent and versioned, _i.e._, which will never be overwritten
   and which their version allows to uniquely identify

## Capitalization on data processing software
We capitalize on the (source code of the) software used to process data,
rather than on the prepared data sets. The software project is instantiated
from a template (_e.g._, with
[Cookiecutter](https://github.com/cookiecutter/cookiecutter)/[Cruft](https://cruft.github.io/cruft/))
and managed through a Git repository.
The Git repository may be audited, including the level of compliance
to the (evolutions of the) template.

## End-to-end data responsibility
Data domain data engineering teams are responsible for the (quality
and service level agreements of the) delivered data sets.
That responsibility includes checking (and potentially fixing) the quality
of the source data sets. As an illustration,
if [medalion](https://www.databricks.com/glossary/medallion-architecture)
(silver/gold/insight) data sets would be manufactured cars,
the responsibility encompasses the quality of every single part
(_e.g._, tires, windshield).
The data engineering teams cannot deflect their responsibility on the quality
of the silver/gold/insight data sets to the quality of the source data sets:
they have to fix the quality of the source data sets if needed.

