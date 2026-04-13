---
title: "SOAP Blueprint, or How I Learned to Stop Worrying and Love WSDLs"
date: 2026-04-13
permalink: /posts/2026/04/soap-blueprint-wsdl-generator-for-sap/
tags:
  - SOAP
  - WSDL
  - SAP
  - API
  - Integration
  - Integration Suite
  - Streamlit
  - Python
---

If you have ever had to write a WSDL file by hand, you know that the experience sits somewhere between filling in a tax form and defusing a bomb. One misplaced namespace, one wrong [nesting style](https://www.oracle.com/technical-resources/articles/java/design-patterns.html), and the consuming system rejects the whole thing with an error message that tells you absolutely nothing useful. If that consuming system happens to be SAP, the error messages may get even more cryptic and they come individually, even though [there are ways](https://me.sap.com/notes/0003599174) of knowing where the problems lay. Furthermore, the documentation on what WSDL structure SAP actually expects is a mixture of tribal knowledge and a bunch of SAP Notes.

(By the way, sorry for the movie title [reference](https://www.imdb.com/title/tt0057012/): I've been revisiting Kubrick lately).

I ran into this problem often enough that I built a small tool to solve it a while ago. It worked, but it had some serious limitations that eventually caught up with me. So I rewrote it from scratch. The result is [SOAP Blueprint](https://soapblueprint.streamlit.app), and I think it turned out rather well. See [GitHub repo](https://github.com/malmriv/soapblueprint) for details.

![A view of SOAP Blueprint](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/soapblueprint-view-tool.png?raw=true)

## What it does

SOAP Blueprint is a web app where you define the fields of your SOAP service (request and response, with names, types, cardinality, and as many levels of nesting as you need), and it generates a WSDL that is ready to use. The resulting WSDL structure is not something I decided: the template has been validated and tested thoroughly with SAP S/4HANA, ECC, Integration Suite and (with the help of some colleagues) MuleSoft.

Beyond the WSDL itself, the tool generates a few other things that I kept wishing I had in previous projects:

- A standalone XSD. Sometimes you just need the schema. The tool extracts it with the correct namespace declaration so you can use it directly.
- Sample SOAP messages for both request and response. These include the full Envelope with the correct namespace handling. Useful for quick tests or for handing to someone who needs to see what the payload looks like. Oh, and the desired data types result in appropriate dummy data! You're welcome :)

![Options available after describing the desired structure](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/soapblueprint-options.png?raw=true)

- A Postman collection, ready to import. This one deserves a bit of context. Postman is notoriously bad at converting WSDLs into working collections. It tends to set the Content-Type to `application/xml` (which is not always accepted), and the generated requests often lack the SOAP Envelope or get the namespace wrong. The collection that SOAP Blueprint generates comes with the correct POST method, `text/xml` as Content-Type, the full Envelope in the body, and a `{{SOAPServiceURL}}` variable so you just need to fill in the endpoint.

![Postman takes the generated collections with no issue :)](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/soapblueprint-postman.png?raw=true)

(Lately, by the way, I have taken to like [Bruno](https://www.usebruno.com) better than Postman for a bunch of reasons. Mainly, the ownership of your requests in your own storage & the blat-free interface. Because Bruno accepts Postman collections directly, the same files can be imported to Bruno directly.)

![Bruno can import the request collections no problem](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/soapblueprint-bruno.png?raw=true)

Field types are drawn from the [W3C XSD built-in datatypes](https://www.w3.org/TR/xmlschema-2/#built-in-datatypes) specification. I left out the more exotic ones (there are types for year-month combinations, for hex-encoded binary data, for things I honestly cannot imagine a use case for). The ones included cover everything I have ever needed: strings, integers, decimals, booleans, dates, datetimes, and a few others.

## What was wrong with the old version

The [previous tool](https://github.com/malmriv/WSDLGen) was written in R using [Shiny](https://shiny.posit.co). It worked for simple cases, but had two problems that made it increasingly painful to use and impossible to maintain:

1. It only supported one level of nesting. A field could have children, but those children could not have children of their own. This sounds like a minor limitation until someone needs an order with line items where each line item has a breakdown of taxes and discounts. That is three levels deep, and it is not unusual to go further. The reason for the limitation was not conceptual (XSD obviously allows arbitrary nesting) but practical: the way I had implemented the tree editing in Shiny made it very hard to generalise beyond one depth. Shiny's reactive model re-evaluates dependencies whenever state changes, and managing a recursive data structure inside that turned into a nightmare of circular updates and broken widget IDs.

2. The code was a single monolithic file. UI callbacks and XML string concatenation lived side by side. There were no tests, no separation of concerns, and no way to change one thing without risking everything else. I tried to fix the nesting problem a few times and gave up each session because understanding the code required loading the entire file into my head. As often happens with Shiny apps (or at least with *my* Shiny apps), what started as a quick personal project ended up being used by colleagues, and by that point refactoring without a testing environment felt like a bad game of [Jenga](https://en.wikipedia.org/wiki/Jenga).

So I have spent a few weeks rebuilding it using [an approach that has worked well for me](https://github.com/malmriv/integration-suite-explorer) in the past. I shamelessly declare myself a [Streamlit](https://streamlit.io) enjoyer.

## How it works now

The core idea that makes the rewrite work is a recursive data model:

```python
@dataclass
class Field:
    name: str
    type: str = "string"
    min_occurs: int = 0
    max_occurs: int | str = 1  # int or "unbounded"
    children: list["Field"] = field(default_factory=list)
```

A field is complex if it has children. The WSDL builder recurses into children and generates the `xsd:complexType` / `xsd:sequence` wrappers automatically. No special cases and no depth limit. The entire generation module is about 80 lines of code using `lxml`, which gives you well-formed XML by construction as well as proper namespace prefixes and pretty-printing without effort.

The UI is a [Streamlit](https://streamlit.io) app. I have used the Python + Streamlit combination [before](https://github.com/malmriv/integration-suite-explorer) with good results, so I already knew the framework's strengths and quirks. The important architectural decision was to keep the core module completely independent of Streamlit: it takes a configuration object and two lists of field trees, and returns strings. The Streamlit app is just a frontend that lets you build those trees interactively. This means I could add a CLI mode or an API endpoint in the future without touching the generation logic at all.

## Lessons learned

1. Starting with the data model instead of the UI changes everything. The old version's UI drove the architecture, and the architecture ended up being "whatever Shiny's reactive model made convenient." The new version's architecture drove the UI, and the UI ended up being a thin layer that translates between session state and `Field` trees. When I needed to make the operation element names configurable (they used to be hardcoded), I changed the dataclass, updated the builder, ran the tests, and then added two text inputs in the Streamlit app. The old version would have required touching every function that concatenated XML strings, with no tests to tell me if I broke something.

2. Do not fight the framework's execution model; embrace it. Streamlit reruns the entire script on every interaction. If you try to make it behave like a traditional stateful GUI (persisting widget state implicitly, reacting to individual changes), you will have a bad time. If you accept that every click is a fresh execution and design your state management around that assumption, things fall into place. Each node in the tree gets a UUID and all widget keys incorporate it. States are explicit rather than "reactive" (something which made maintaining a Shiny app a bit of a nightmare). It is not the most elegant approach in the universe, but it works and sometimes that's good enough.

3. Separating core logic from the UI is what makes the tool maintainable months later. The old version was proof of what happens when you skip this: I was unable to maintain it. The new version has a test suite that covers all the cases (main and edge) that I could think of and has a good separation of concerns ([even if some devs think there are better approaches](https://grugbrain.dev/#grug-on-soc)). Adding the Postman collection feature took about an hour because the sample message generator already existed in the core, and all I needed was a function that wrapped it in the right JSON structure. In the old version, adding a feature like that would have meant surgery on the monolith.

## Try it

The tool is deployed on [Streamlit Community Cloud](https://soapblueprint.streamlit.app). The source is on [GitHub](https://github.com/malmriv/soapblueprint). If you work with SOAP services in SAP (or anywhere else, really), give it a try. If the generated WSDL does not work with your system, I would genuinely like to know, along with any trace that allows me to find out why.
