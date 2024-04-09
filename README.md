# FAGTSY

## Introduction

FAGTSY is an architecture (in development) composed of 5 solutions and 2 databases aiming to create a website providing customized content to users based on parameters inputted (the actual topic won't be disclosed until the website is up and running).

## Architecture

Given the nature of the information 2 databases are needed:

- A relational database for technical information
- A graph database for descriptive information

Following a brief description of the solutions:

- 1 Feeder (Relational): a microservice architecture in charge of populating the relational database with raw technical information ([DBFeeder](https://github.com/dapalex/DBFeeder) is a simplified version of the solution)
- 1 projector: a component in charge of projecting/transforming the raw information to a normalized version that will be primarily available
- 1 Feeder (graph): a microservice architecture (DBFeeder inspired) in charge of populating the graph database with the descriptive information
- 1 web API: client of the databases and server of the web application
- 1 web application: the publicly accessible website for the users

Following an overview of the architecture

![Architecture](https://github.com/dapalex/FAGTSY/blob/main/Docs/Architecture.png)


### The Feeders - Design

The Feeders are in charge of populating the databases.
They work in 2 phases:

- Initial: ran before the website will be published, a massive population
- Online: while the website will be up and running, it will add information in case of requests not satisfied

A feeder is composed of at least 3 containerized .Net 7 service workers communicating via an event bus (RabbitMQ):

- Crawler: fetching the webpages with the information
- Scraper: scraping the webpage and preparing the information to be written
- DAC: populating the DB with the information from the scraper

The Initial phase gets the information through configuration files leading the crawling order. The Online phase works when a request has "failed" (the information requested by the user is not available), it gets the information relating to a single item.

A simplified version is visible [here](http://github.com/dapalex/DBFeeder).

### Projector

The role of the projector is to:

- Normalize the information
- Unify the information from different sources
- Linking the technical information to domain tables
- Finally, rank the elements based on different conditions


### The web API

The web API acts as bridge between the databases (as reader) and the web application: it retrieves and organize the information requested.
Additionaly, in case of failed request, it activates the Feeders in order to update the databases with the missing information. 
The API is an ASP .Net Core API in .Net 7 composed of the following projects:

- API: .Net Web API interface exposing the endpoints
- BL: .Net library elaborating the information retrieved from the database to customized version for the client

Following an overview of the design

![API Design](https://github.com/dapalex/FAGTSY/blob/main/Docs/APIDesign.png)

### The web application

The web application is the only solution exposed publicly, it is the actual website available on internet.
Through its UI the users can request information in 2 ways:

- Find: inputting technical information the user can find out the best match
- Search: through a textbox the user can request information on a particular item

The application is a ASP .Net Core Web Application in .Net 7 for the backend and Angular 15 for the frontend.
