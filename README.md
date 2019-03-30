# FHIR storage & analytics

https://github.com/niquola/FHIR-Kiev-March-2019-slides

Nikolai Ryzhikov (gh/tw/tg @niquola)

CTO @ Health Samurai








## Health Samurai

* Aidbox - FHIR backend
* Fhirbase - OS database for FHIR
* A lot of OS










##  1. Agenda

1. How to persist FHIR data?
2. How to search FHIR data?




## 2. Use Cases

1. Legacy system
2. Brand new FHIR system




## 3. FHIR Facade (legacy)

Provide FHIR API to your existing data




## 4. FHIR API

read: GET /Patient/pt-id
search: GET /Patient?name=Ivan

create: POST /Patient
update: PUT /Patient/pt-id
update: DELETE /Patient/pt-id

transaction: POST /

demo with aidbox



## 5. Legacy Facade

TODO:

1. Map legacy schema to FHIR
2. Map FHIR operations to legacy queries 


Levels:

1. READONLY API
2. WRITE API


Architecture:

1. [System] <-(transform)-> API...
2. [System] <-> [FHIR server] <-> API...

## 6. FHIR Mapping

Patient

### Legacy <=> FHIR

* last_name       <=> name.0.family
* first_name      <=> name.0.given.0
* ssn             <=> identifier.{system: 'ssn'}.value
* driver_license  <=> identifier.{system: 'dl'}.value
* bod             <=> birthDate


### FHIR <=> Legacy

```yaml

resourceType: Patient
name:
- given: 
  - $ .first_name
  family: $ .last_name
identifier:
- system: ssn
  value: $ .ssn
- system: dl
  value: $ .driver_license
birthDate: $ .bod

```

jute  demo: https://jute-demo.aidbox.app/index.html


FHIR mapping language - https://www.hl7.org/fhir/mapping-language.html

BX mapping by me - comming soon!



## 7. Map FHIR Operations

```http
GET /Patient?name=ivanov

```


```sql
SELECT * 
 FROM legacy_patient
WHERE last_name ilike 'ivanov%'
   OR first_name ilike 'ivanov%'
```

Easy for simple cases, 
Can get too complicated


##  8. WRITE FHIR BACK

FHIR is too flexible

* more than you need 
* less than you need

Profile FHIR!

* put constraints
* fix terminology
* define extensions

Welcome to wonderful world of PROFILING!





## 9. FHIR-FHIRST  System

You want to store all FHIR information!




## 10. Nature of FHIR data

* Hierarchical
* Extensible (https://www.hl7.org/fhir/extensibility.html)

See FHIR pt ...




## 11. Options

* Tabular
* Document
* Hybrid




## 12. Tabular

We can generate tables 
based on FHIR metadata

```
create table patient (id, birthDate);
create table patient_name (family text, given text[]);
create table patient_identifier (family text, given text[]);
create table patient_extensions (path text[], value ...);
```

We have about 200 resources
and will got xK tables.





## 13. Tablar trade-offs

JOINS! kill the performance

* hierarchiecal structure
* extensibility


## 13. HAPI & Vonk


## 14. Document


Store FHIR in JSON as is!

* MongoDB; CouchDB; Elastic
* Easy CRUD and simple Search
* Clusters out of the box




## 15. Document trade-offs 

* - ACID
* - SQL





## 16. Hybrid

Store FHIR as is in relational databases

* jsonb in pg
* json in mysql, oracle, mssql
* protobuf in bigquery
* struct in spark





## 17. Fhirbase & Aidbox

JSONB in postgresql

* fast access attributes
* jsonb functions
* indexing support

See our masterclass from pgconf - https://github.com/fhirbase/master-class

## 18. Fhifrbase & Aidbox

```sql
create table patient (id text, ts timestamp, resource jsonb);

select * 
  from patient
 WHERE resource#>>'{name,0,famiy}' ilike 'ivanov%' 
   AND resource @> '{"identifier": [{"system": "ssn", "value": "?"}]}'
```


https://fbdemo.aidbox.app/
You can mix SQL & JSON

```sql
SELECT p.resource#>'{name,0,family}', count(e.id) 
 FROM patient p
 join encounter e
   on e.resource#>>'{subject,id}' = p.id
group by p.id, p.resource#>'{name,0,family}'
order by count(e.id) desc
LIMIT 10;
```

demo fhirbase

## 19. SQL on FHIR

https://github.com/FHIR/sql-on-fhir/blob/master/sql-on-fhir.md

Storage format

* references 
* union types 
* first class extensions

## 20. Refs

```yaml

rt: Encounter
subject:
  reference: Patient/pt-1

subject:
  resourceType: 'Patient'
  id: pt-1
  patient_id: pt-1
```

## 21. Union

Observation.value[x]


```yaml

rt: Observation
valueQuantity:
  value: 180
  units: cm


rt: Observation
value:
  Quantity:
    value: 180

rt: Observation
valueCoding:
  - code: 'loinc'

rt: Observation
value:
  Coding:
    code: loinc
```

Observation.value is null

## 22. Extensions

```yaml
rt: Patient
extension:
- url: '.../race'
  valueCode: 'white'

rt: Patient
race: 'white'

```

## 23. Indexing

```
rt: Observation
type: 
- system: loinc
  code: ...
- system: snomed
  code: ...

rt: Observation
type: 
  loinc: ...
  snomed: ...
  
  
rt: Patient
identifier:
- system: ssn
  value: ...
- system: driver
  value: ...

rt: Patient
identifier:
  ssn: ...
  driver: ...
```

## 24. Welcome

* SPb FHIR meetup 4 april
  * https://t.me/FHIRmeetups

* Montreal May 
  * Bulk API & S&A together as a Population Track

* Redmond June
  * S&A tutorial

* Moscow September
  * FHIR starter
