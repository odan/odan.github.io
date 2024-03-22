---
title: ASP.NET - Streamlining OpenAPI Documentation
layout: post
comments: true
published: true
description: 
keywords: aspnet, csharp, dotnet, openapi, swagger, documentation
---

## A Web-Centric Approach

The ASP.NET Core team [moves away from Swashbuckle](https://github.com/dotnet/aspnetcore/issues/54599) 
creates room for creativity as the framework develops, 
especially in how it handles API documentation and integrates OpenAPI (previously Swagger). 
Since `Microsoft.AspNetCore.OpenApi` will be improved in.NET 9 with a focus toward document generation, 
other approaches have a chance to adopt and expand OpenAPI support. 
In the middle of this shift, I offer a different approach for integrating 
Swagger OpenAPI that makes use of the web's simplicity and universality: 
Supplying an HTML file (Swagger-UI) and a YAML file that are rendered 
in the browser to show the OpenAPI specification.

## Swagger-UI

Swagger UI is a collection of HTML, JavaScript, and CSS assets that dynamically 
generate beautiful documentation from a OpenAPI specification (Yaml file)
without requiring the implementation of its server logic.

![Swagger UI](https://github.com/odan/odan.github.io/assets/781074/c828a84b-65dc-43f3-98fb-471f96c360ae)

## Using HTML and YAML for OpenAPI Documentation

This methodology is centered around the creation and distribution of two core elements:

**Swagger-UI HTML Page:** A static HTML page that uses Swagger-UI,
an interactive API documentation web tool that renders OpenAPI specifications
into a user-friendly and interactive interface.

**OpenAPI Specification YAML File:** The API's OpenAPI specification,
defined in YAML format, serves as the source of truth for the API documentation.

## Benefits of the HTML/YAML Approach

Besides the accessibility, there are even more benefits such as:

**Enhanced Change Management:** By relying on static assets like HTML and YAML, 
this approach minimizes the complexity involved in API documentation, 
making it easier to maintain and update.

The process of documenting your API and making updates to its specification 
is now handled directly within the YAML file for each version of your API. 
This means that the OpenAPI schema for your API 
(including descriptions of endpoints, parameters, and responses) 
will be manually written and updated in the YAML file, 
rather than being automatically generated from the source code. 

This approach allows for greater flexibility and control over the API documentation, 
but it requires that changes to the API are carefully reflected in the YAML file 
to ensure accuracy and consistency of the documentation. 

It's essential to keep the documentation up to date with any modifications 
or additions to the API to provide users with a reliable and comprehensive 
understanding of the API's capabilities.

Additionally, by incorporating the OpenAPI YAML 
file into your version control system, you can precisely track and 
manage changes over time. This setup enhances 
collaboration and change management, allowing for a clear history 
of modifications and facilitating easier rollback if needed.

To modify a YAML file, you can use a variety of tools ranging from 
[Swagger Editor](https://editor.swagger.io/) 
to more sophisticated editors like [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer).

**Flexibility:** Developers can easily customize the Swagger-UI page 
to fit their branding or add additional functionality, 
offering a tailor-made documentation experience.

**Portability:** The YAML file can be distributed independently of the application,
allowing for easy sharing, version control, and use in other tools or environments.

By managing your API documentation through YAML files and a static Swagger UI, 
you have the flexibility to host your documentation on any server or platform 
that supports static content. 

This approach is not limited to your ASP.NET Core application's server; 
you can easily deploy the same set of documentation files 
(index.html and the YAML OpenAPI schema) on external servers, 
cloud storage services, or even on GitHub Pages.

**💡 Note:** I know, this method does not aligns with the direction of ASP.NET Core team,
but it offers a practical and efficient solution to the challenges of API documentation 
in the evolving landscape of web development.

## Setup

To set up a directory structure for hosting Swagger UI and your OpenAPI schema in an 
ASP.NET Core project, follow these instructions. 

**Create the wwwroot Folder**

If your ASP.NET Core project does not already have a `wwwroot` folder at its root, create one. 
This folder is the web root directory and serves as the base path for your web application's static files.

**Add a docs/v1 Subfolder**

Inside the `wwwroot` folder, create a new folder path `docs/v1`. 
This path will host the Swagger UI and your OpenAPI schema for version 1 of your API.

**Add index.html for Swagger UI**

Inside the `docs/v1` folder, create a file named `index.html`. This HTML file will include the Swagger UI setup. 

Copy / paste the following source-code into the `index.html` file:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>API Specification - Swagger UI</title>
    <link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/5.12.0/swagger-ui.min.css">
</head>
<body>
    <div id="swagger-ui"></div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/js-yaml/4.1.0/js-yaml.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/5.12.0/swagger-ui-bundle.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/5.12.0/swagger-ui-standalone-preset.min.js"></script>
    <script>
        window.onload = async function () {
            try {
                const response = await fetch('openapi.yml');
                if (!response.ok) {
                    throw new Error('Network response was not ok.');
                }
                const yaml = await response.text();
                const ui = SwaggerUIBundle({
                    spec: jsyaml.load(yaml.trim()),
                    dom_id: '#swagger-ui',
                    deepLinking: true,
                    supportedSubmitMethods: [],
                    presets: [
                        SwaggerUIBundle.presets.apis,
                    ],
                    plugins: [
                        SwaggerUIBundle.plugins.DownloadUrl
                    ],
                });
                window.ui = ui;
            } catch (error) {
                alert("Error loading YAML file: " + error.message);
            }
        }
    </script>
</body>
</html>

```

This HTML file serves to display an API specification using Swagger UI, 
a tool that renders OpenAPI specifications into interactive API documentation. 

Upon loading, the file fetches an API schema from a `openapi.yml` file using a HTTPS network request, processes the YAML content to JSON 
using the `js-yaml` library, and then initializes the Swagger UI with this
processed specification. 

The HTML includes links to CSS for styling and JavaScript libraries 
for Swagger UI functionality and YAML processing.

**Add Your OpenAPI Schema YAML File**

Within the `docs/v1` directory, create your OpenAPI schema file with 
a `.yml` extension (e.g., `openapi.yml`). This YAML file contains your API's 
specification in the OpenAPI format. 

Copy the following sample API specification or use your own file:

```yaml
openapi: 3.0.0
info:
  title: Sample API
  description: Optional multiline or single-line description in [CommonMark](http://commonmark.org/help/) or HTML.
  version: 0.1.9

servers:
  - url: http://api.example.com/v1
    description: Optional server description, e.g. Main (production) server
  - url: http://staging-api.example.com
    description: Optional server description, e.g. Internal staging server for testing

paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in CommonMark or HTML.
      responses:
        '200':    # status code
          description: A JSON array of user names
          content:
            application/json:
              schema: 
                type: array
                items: 
                  type: string
```

Example project:

![project](https://github.com/odan/odan.github.io/assets/781074/ca3035b8-81ce-4ee2-8332-a0caba793c8b)

To serve the static files you've placed in the `wwwroot` folder and its 
subdirectories, you'll need to enable the static files middleware in your 
ASP.NET Core application. This is done by adding `app.UseStaticFiles();` 
to the Configure method in your `Startup.cs` or `Program.cs` file. 

```cs
// ...

app.UseDefaultFiles();
app.UseStaticFiles(new StaticFileOptions
{
    ServeUnknownFileTypes = true
});

// app.MapControllers();

app.UseRouting();

// ...
```

**Notice:** The StaticFiles middleware should be added before the Routing or Controllers middleware.

With the setup complete, you can access your Swagger UI by navigating to 
`https://<your-host>/docs/v1/index.html` or just `https://<your-host>/docs/v1/`
in a web browser. This will render your API documentation using Swagger UI, 
allowing you to interact with your API’s endpoints directly.

**Preview**

![swagger](https://github.com/odan/odan.github.io/assets/781074/cd5c106c-f918-4e12-b7f7-b5737442850e)


## New API Versions

With this YAML-based documentation approach for your Swagger UI, 
it becomes possible to adopt a "design-first" methodology for your APIs. 
This means you can design and prototype new APIs or endpoints without 
writing any actual code (such as actions or controllers) upfront. 
You can outline the entire API structure, define endpoints, parameters, 
and responses directly within the YAML file. This enables you to present 
a comprehensive proposal of the new API version or changes to 
stakeholders or customers for discussion and approval before any development starts.

When introducing a new version (e.g., v2) of your API, 
follow these steps to ensure your Swagger UI documentation reflects the updated version:

* Inside the `wwwroot/docs` folder, create a new subfolder named `v2` to host the Swagger UI 
and the OpenAPI schema for the new version of your API.
* Place a new `index.html` file within the `docs/v2` directory with the same contents.
* Create a new YAML file within `docs/v2` that contains the OpenAPI schema for API version 2. 
This file should reflect all the new endpoints, parameters, responses, and any other changes or additions specific to version 2.
* No additional changes are required if you're already serving static files with the StaticFiles middleware.
* You can access the version 2 documentation by navigating to `https://<your-host>/docs/v2/index.html` in a web browser.

## Conclusion

This setup organizes your API documentation neatly within your ASP.NET Core project, 
making it easily accessible while keeping it separate from your application logic
and safe from the next breaking-changes in the .NET ecosystem.
