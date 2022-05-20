# Document Generation using Accord Project Templates

This project illustrates the power of using the Accord Project template stack
for **type safe** document generation.

> By **type safe** we mean that all variables referenced by the template are correct
and that any errors in the conditional logic of a template are detected
during authoring time, ensuring that there cannot be runtime errors during 
template evaluation.

## Use Case

This scenario is relatively representative of the type of conditional text generation that
people often have to perform during document generation.

In this scenario we define a data model for a `Patient`, and then create a document generation
template that refers to `Patient` properties. We show how to create the model, template, conditional logic
and formulae that allow you to generate a document from a JSON data payload which describes a given patient.

In addition we show how to use the Accord Project markdown-transformation stack to transform the output
of document generation to various output formats, including JSON, markdown, PDF and HTML.

## Data Model

The data model is defined using the Concerto modelling language in [model.cto](./model/model.cto). We define a `PatientClause` with some required and optional properties — a mixture of primitive and complex types.

## Template (Grammar)

The template is defined using TemplateMark in [grammar.tem.md](./text/grammar.tem.md). The template contains:
1. Formatted text, using Markdown syntax
2. Variables using the `{{name}}` syntax
3. Formatted locale specific date variables using the `{{dob as "DD/MM/YYYY"}}` syntax to format a `DateTime`
4. Conditional inclusion of text using `{{#if}} ... {{else}} ... {{/if}}` blocks
5. Expansion of ordered lists using `{{#olist allergies}}` blocks
6. Calling a user defined function and formatting the result as a monetary amount using `{{% procedureCost(contract) as "K0,0.00"%}}`
7. Defining a function inline to perform complex string concatenation based on data

## Template Logic (Functions)

Expressions and functions are defined using the (type-safe) Ergo functional programming language.

The Ergo functions are defined in [logic.ergo](./logic/logic.ergo).

## Template Metadata

Templates are packaged as zip files and must include a [package.json](./package.json) manifest file, which includes name, description, version, keywords etc that make the template searchable within a template library.

## Install Tools

```
npm i -g @accordproject/cicero-cli
npm i -g @accordproject/markdown-cli
```
## Draft

Converts JSON data to text via a template. Using data:

```
cat data-allergies.json                                                       
{
    "$class": "demo.PatientClause",
    "name": "Matt Roberts",
    "dob": "1975-03-20T00:00:00.000+01:00",
    "procedure": "teeth whitening",
    "clauseId": "bfd5d165-c3ab-4cb1-91ac-2cda250aeb0c",
    "hasAllergies" : true,
    "address" : {
      "country" : "USA",
      "state" : "ME"
    },
    "allergies" : [
      {
        "$class" : "demo.Allergy",
        "name" : "Paracetamol",
        "cost" : {
          "$class" : "org.accordproject.money.MonetaryAmount",
          "doubleValue" : 25.5,
          "currencyCode" : "GBP"
        }
      },
      {
        "$class" : "demo.Allergy",
        "name" : "Hay Fever",
        "cost" : {
          "$class" : "org.accordproject.money.MonetaryAmount",
          "doubleValue" : 45.8,
          "currencyCode" : "GBP"
        }
      }
    ]
  }
```

Produces output:

```
cicero draft --data data-allergies.json  --output draft.md --unquoteVariables 
14:26:26 - INFO: Using current directory as template folder
14:26:27 - INFO: Creating file: draft.md
14:26:27 - INFO: Patient Acceptance
----

The patient Matt Roberts born on 19/03/1975 hereby accepts the procedure teeth whitening.

Patient has declared the following allergies:
1. Paracetamol
2. Hay Fever

The cost of the procedure will be: £171.30.

Each party hereby irrevocably agrees that process may be served on it in
any manner authorized by the Laws of the state of ME, USA.
```

Note that the allergy conditional text was included, the allergy array was expanded, the procedure cost was calculated via a formula and then state of law paragraph was conditionally assembled.

Now for a patient without allergies:

```
cat data.json                                                                 
{
    "$class": "demo.PatientClause",
    "name": "Matt Roberts",
    "dob": "1975-03-20T00:00:00.000+01:00",
    "procedure": "teeth whitening",
    "address" : {
      "country" : "UK",
      "state" : "Oxford"
    },
    "hasAllergies" : false,
    "allergies" : [
    ],
    "clauseId": "bfd5d165-c3ab-4cb1-91ac-2cda250aeb0c"
  }%        
```

```
cicero draft --data data.json --output draft.md --unquoteVariables 
14:30:10 - INFO: Using current directory as template folder
14:30:10 - INFO: Creating file: draft.md
14:30:10 - INFO: Patient Acceptance
----

The patient Matt Roberts born on 19/03/1975 hereby accepts the procedure teeth whitening.

Patient has no declared allergies.

The cost of the procedure will be: £100.00.

Each party hereby irrevocably agrees that process may be served on it in
any manner authorized by the Laws of the county of Oxford, UK.
```

Finally, for a patient without allergies and without an address:

```
cat data-no-address.json 
{
    "$class": "demo.PatientClause",
    "name": "Matt Roberts",
    "dob": "1975-03-20T00:00:00.000+01:00",
    "procedure": "teeth whitening",
    "hasAllergies" : false,
    "allergies" : [
    ],
    "clauseId": "bfd5d165-c3ab-4cb1-91ac-2cda250aeb0c"
  }
```

```
cicero draft --data data-no-address.json --output draft.md --unquoteVariables  
14:31:39 - INFO: Using current directory as template folder
14:31:40 - INFO: Creating file: draft.md
14:31:40 - INFO: Patient Acceptance
----

The patient Matt Roberts born on 19/03/1975 hereby accepts the procedure teeth whitening.

Patient has no declared allergies.

The cost of the procedure will be: £100.00.

Each party hereby irrevocably agrees that process may be served on it in
any manner authorized by the Laws of the county of London, UK.
```

Note that governing law defaults to `London, UK`.

## Format Conversion

To convert output to JSON tree:

```
markus transform --from markdown --to ciceromark --input draft.md --output draft.json
```

```
cat draft.json 
{"$class":"org.accordproject.commonmark.Document","xmlns":"http://commonmark.org/xml/1.0","nodes":[{"$class":"org.accordproject.commonmark.Heading","level":"2","nodes":[{"$class":"org.accordproject.commonmark.Text","text":"Patient Acceptance"}]},{"$class":"org.accordproject.commonmark.Paragraph","nodes":[{"$class":"org.accordproject.commonmark.Text","text":"The patient Matt Roberts born on 19/03/1975 hereby accepts the procedure teeth whitening."}]},{"$class":"org.accordproject.commonmark.Paragraph","nodes":[{"$class":"org.accordproject.commonmark.Text","text":"Patient has no declared allergies."}]},{"$class":"org.accordproject.commonmark.Paragraph","nodes":[{"$class":"org.accordproject.commonmark.Text","text":"The cost of the procedure will be: £100.00."}]},{"$class":"org.accordproject.commonmark.Paragraph","nodes":[{"$class":"org.accordproject.commonmark.Text","text":"Each party hereby irrevocably agrees that process may be served on it in"},{"$class":"org.accordproject.commonmark.Softbreak"},{"$class":"org.accordproject.commonmark.Text","text":"any manner authorized by the Laws of the county of London, UK."}]}]}% 
```

To convert output to PDF:

```
markus transform --from markdown --to pdf --input draft.md --output draft.pdf
```

To convert output to HTML:

```
markus transform --from markdown --to html --input draft.md --output draft.html
```

## Parse

One of the unique characteristics of the Accord Project template stack is that it can
function in **BOTH DIRECTIONS** either generating text from a template + JSON, **OR** 
extracting JSON from text + template.

Parsing the following markdown:

```
cat text/sample.md 
## Patient Acceptance

The patient "Dan Selman" born on 30/12/1970 hereby accepts the procedure "tonsil removal".

Patient has no declared allergies.

The cost of the procedure will be: {{% COMPUTED %}}.

Each party hereby irrevocably agrees that process may be served on it in
any manner authorized by the Laws of {{% COMPUTED %}}.
```

Extracts the JSON object:

```
cicero parse
14:34:44 - INFO: Using current directory as template folder
14:34:44 - INFO: Loading a default text/sample.md file.
14:34:45 - INFO: {
  "$class": "demo.PatientClause",
  "name": "Dan Selman",
  "dob": "1970-12-30T00:00:00.000+01:00",
  "procedure": "tonsil removal",
  "hasAllergies": false,
  "allergies": [],
  "clauseId": "739b55f0-f9d0-4c9c-9d27-02de1424c257",
  "$identifier": "739b55f0-f9d0-4c9c-9d27-02de1424c257"
}
```

Versus:

Parsing: 

```
cat ./text/sample-allergies.md 
## Patient Acceptance

The patient "Dan Selman" born on 30/12/1970 hereby accepts the procedure "tonsil removal".

Patient has declared the following allergies:
1. "Paracetamol"
2. "Hay Fever"

The cost of the procedure will be: {{% COMPUTED %}}.

Each party hereby irrevocably agrees that process may be served on it in
any manner authorized by the Laws of {{% COMPUTED %}}.
```

Extracts the JSON object:

```
cicero parse --sample ./text/sample-allergies.md
14:39:27 - INFO: Using current directory as template folder
14:39:28 - INFO: {
  "$class": "demo.PatientClause",
  "name": "Dan Selman",
  "dob": "1970-12-30T00:00:00.000+01:00",
  "procedure": "tonsil removal",
  "hasAllergies": true,
  "allergies": [
    {
      "$class": "demo.Allergy",
      "name": "Paracetamol"
    },
    {
      "$class": "demo.Allergy",
      "name": "Hay Fever"
    }
  ],
  "clauseId": "cf305860-8353-42f4-831f-f17ec208617f",
  "$identifier": "cf305860-8353-42f4-831f-f17ec208617f"
}
```

