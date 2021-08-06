---
layout: post
title:  "Export Postman Collection To Swagger"
date: 2021-08-06
tags: [api, laravel, postman, swagger]
---

If I'm not mistaken, Postman is the most popular REST client out there. It's used in testing API endpoints which can be grouped into project based collections. Getting these collections out to be shared with teams(internal/external) in a clean format can be a pain sometimes. Swagger is my preferred way of sharing my API documentation with my colleagues and teams.
<!--more-->

## What is API Documentation?
API documentation is technical content that documents the API. It includes instructions on how to effectively use and integrate the API. It also provides updates on the API’s lifecycle such as new versions or retirement. Some aspects of API documentation can be generated automatically via Swagger or other documents.

## Why document APIs?
Even though working with Agile, and one of the principles being “Working Software Over Comprehensive Documentation”, from Agile Manifesto, documentation matters, and we invest our time doing it. Why? Take a look to the following points:

- Improves the experience for developers consuming an API;

- Decreases the amount of time spent on-boarding new users (internal developers or external partners). New users will start being productive earlier and will not depend on a person (already with the knowledge) who would need to spend slots of their time to explain how the API is design and how it works;
    
- Leads to good product maintenance and quicker updates. It helps your internal teams know the details of your resources, methods, and their associated requests and responses, making maintenance and updates quicker;
    
- Agreement on API specs for the endpoints, data, types, attributes and more. And this helps internal and external users to understand the API and know what it can do;
    
- The API documentation can act as the central reference that keeps all the team members aligned on what your API’s objectives are, and how the API’s resources are exposed;

- Enables bugs and issues identification in the API’s architecture when defining it with the team;

Exporting PostMan Collection
In testing APIs, I use [PostMan](https://postman.io) to test and document my APIs by creating a collection under a namespace with PostMan. For more on how to create a Postman collection, see [here](https://learning.postman.com/docs/getting-started/creating-the-first-collection)

## Generating A Postman Collection from Laravel
To achieve this, we will need to add a package to our project which automatically generate a Postman collection based on the routes we have in the project.

1. We pull in the package via Composer;

```bash
composer require profclems/postman-collection-generator --dev
```

This installs the package only on our development or local instances. Once done, we specify the route we are interested in, ie. 

`routes/api.php` or `route/web.php`.

2. We generate a Postman collection

```bash 
php artisan postman:collection:export NameForCollection --api
```

The above command will execute the export of a Postman collection our API routes with the specified name `NameForCollection`. The generated collection is located in `storage/app` folder.

Should we want to generate a Postman collection based of our web routes, the command changes to:

```bash
php artisan postman:collection:export NameForCollection --web
```

We can set the base url and port should we desire to, by passing:

```bash 
--url=baseUrlHere
```

to set the base url for the collection and 

```bash
--port=8000
``` 

to set a port for which we will be serving the API.

## Manually Generating A Postman Collection
The Postman documentation details how to achieve this, [here](https://learning.postman.com/docs/getting-started/importing-and-exporting-data/#exporting-postman-data)

## Convert Postman Collection To OpenAPI Standard
With the Postman collection exported as JSON file, we head over to [APITransform](https://apitransform.com/convert) and then fill out a form
{% include image_caption.html imageurl="/images/posts/2021/api-transform.png" title="Form from API Transform" caption="Form from API Transform" %}

We then, proceed to upload our Postman collection for conversion to the OpenAPI standard. The generates a download link to store the converted document (in JSON file format) on our local machine.

## Importing Into Swagger
Next, we hard over to [Swagger](https://swagger.io) and log in to our account. Once logged, we create a new API by clicking new then selecting **Create New API** from the options.

A form is presented to be filled(see below) out with information about our project. Set OpenAPI version to **3.0.0** and choose None from the Template dropdown. Fill out the Name, Owner and Project fields and assign the appropriate visibility option by choosing from the dropdown options. Turn Auto Mock API on. Then click Create API button to submit the form.
{% include image_caption.html imageurl="/images/posts/2021/select-template.png" title="Swagger New API Form" caption="Swagger New API Form" %}


We then paste the contents of the downloaded OpenAPI document into swagger and we are prompted confirm if we want to convert the JSON content to YAML instead. We confirm and the form is auto-saved and is editable so can update by making any changes we want to Project name, base url and API description, etc.
{% include image_caption.html imageurl="/images/posts/2021/open-api-content.png" title="Swagger Form For API Documentation" caption="Swagger Form For API Documentation" %}

Once this is completed, our documentation is completed and ready to be shared with team members or/and the general public.