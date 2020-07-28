<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

# IBM Cloud Functions using Node.js

These examples will show you how to take Node.js functions and deploy and run them within the IBM Cloud Functions (ICF) Serverless platform.  Further examples will show you how to take advantage of some of ICF's built-in features to easily realize the true power of Serverless against some of its top use cases.

If you like using ICF, you can sign-up for the *free*, self-study [Cognitve Class.ai](https://cognitiveclass.ai/) course and get [Acclaim badge](https://www.youracclaim.com/org/ibm/badge/serverless-computing-using-cloud-functions-developer-i) certified:
* [Serverless Computing using Cloud Functions - Developer I course](https://cognitiveclass.ai/courses/serverless-computing-using-cloud-functions-developer-1)

## Index

* [Background](#background)
* [Prerequisites](#prerequisites)
* [Examples](#examples)
    * [Hello world](#hello-world)
        * [Create and list action](#create-and-list-action)
        * [Invoke action](#invoke-action)
            * [Blocking invocation (results only)](#blocking-invocation-results-only)
            * [Blocking invocation (full response)](#blocking-invocation-full-response)
            * [Non-blocking invocation](#non-blocking-invocation)
        * [Invoke with parameters](invoke-with-parameters)
    * [Hello world with parameters](#hello-world-with-parameters)
        * [Update action](#update-action)
        * [Invoke using command line parameters](#invoke-using-command-line-parameters)
        * [Invoke using parameter file](#invoke-using-parameter-file)
    * [ZIP action](#zip-action)- *packaging NPM modules dependencies*
    * [Asynchronous action](#asynchronous-action) - *using Promises*
    * [Web action](#web-action) - *automatically making action output web-accessible using [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)*
    * [Sequencing actions](sequences.md#sequencing-actions) - *easily create sequences of existing actions*

* [Observations](#observations)

## Background

**Actions** are stateless code snippets (functions) that run on the ICF platform. The platform supports functions written in JavaScript (Node.js) as well as many other programming languages such as Python, Swift and Java.

Actions can be used to do many things. For example, an action can be used to respond to a database change, aggregate a set of API calls, post a Tweet, or even work with AI and analytics services to detect objects in an image or streamed video.

The ICF platform is based upon the [Apache OpenWhisk](https://openwhisk.apache.org/) project and implements an [Observer pattern](https://en.wikipedia.org/wiki/Observer_pattern) where **Actions** can be explicitly invoked or run in response to an event which **Trigger** it when connected by a **Rule**. The input to an action and the result of an action are, by default, a JSON dictionary of key-value pairs where the key is a string and the value a valid JSON value. This programming model is shown here:

![Programming model](images/ICF-Programming-Model.png)

Actions can also call other actions or even be composed into sequence of actions.  There are even specialized actions called *Web actions* that are annotated making them publicly accessible to quickly enable you to build web based applications and handle HTTP data directly with any `Content-Type`.

---

## Prerequisites

In order to run these examples, you will need to:

##### Register for and configure a free IBM Cloud Account

1. Register for a free IBM Cloud account using the linked instructions:
    - [https://cloud.ibm.com/registration](https://cloud.ibm.com/registration)

2. Make sure your account targets a region that supports IBM Cloud Functions using the following link:
    * [Target a supported regions](https://cloud.ibm.com/docs/openwhisk?topic=cloud-functions-cloudfunctions_regions) using the CLI.

##### Install the IBM Cloud Command Line Interface (CLI) and Cloud Functions plugin

1. Install the [IBM Cloud Command Line Interface (CLI)](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started) by following the steps outlined on the linked page.

2. Install the [Cloud Functions (CF)](https://cloud.ibm.com/functions/learn/cli) plugin by following the steps outlined on the linked page.
    * *New accounts will automatically have a single `Namespace` (represented by a UUID) and `Resource Group` (called `Default`) created.*

---

## Examples

In ICF, an action represents the serverless function, along with its conventions and metadata. Please note that the word _function_ used synonymously with the term _action_ since actions contain the functional source code, which action metadata is built around.

### Hello world

#### Create and list action

The following steps and examples demonstrate how to create your first JavaScript action.

1. Create a JavaScript file named `hello.js` with this function:

    ```javascript
    function main() {
        return {payload: 'Hello World!'};
    }
    ```

    The JavaScript file might contain additional functions. However, by convention, a function called `main` is the default entry point for the action.

2. Create an action from the `hello.js` JavaScript function, naming it `hello`:

    ```bash
    ibmcloud fn action create hello hello.js
    ```

    ```bash
    ok: created action hello
    ```

3. List all actions. The `hello` action you just created should show:

    ```bash
    ibmcloud fn action list
    ```

    ```bash
    actions
    <NAMESPACE>/hello       private    nodejs10
    ```

    You can see the `hello` action you just created under your account's default `NAMESPACE` and it is, by default, marked `private` with no external endpoint.

#### Invoke action

After you create your action, you can run it on ICF with the `invoke` command using one of two modes:

- **[Blocking](#blocking)** - A blocking invocation request will wait for the activation result to be available.  This is done by specifying the `--blocking` flag on the command line.
- **[Non-blocking](#non-blocking)** - A non-blocking invocation will invoke the action immediately, but not wait for a response.

##### Blocking invocation (results only)

If you wish to see only the function's output `payload`, you can use the `--result` flag or its short form `-r` to perform an implied blocking invocation request.

1. Invoke the `hello` action requesting the results only:

    ```bash
    ibmcloud fn action invoke hello --result
    {
        "payload": "Hello World!"
    }
    ```

##### Blocking invocation (full response)

If you wish to wait for the full HTTP response, simply use the `--blocking` flag.

1. Invoke the `hello` action using the command line as a blocking activation:

    ```bash
    ibmcloud fn action invoke hello --blocking
    ```

    The first line provides a per-invocation *Activation ID* which can be used at any time to lookup the complete *Activation record* which follows.

    ```bash
    ok: invoked /_/hello with id 44794bd6aab74415b4e42a308d880e5b
    ```

    The remaining lines show the complete JSON *Activation record* which contains the response and a `payload` key whose value is the function's result:

    ```json
    ...
    "response": {
          "result": {
              "payload": "Hello World!"
          },
          "size": 26,
          "status": "success",
          "success": true
      },
      ...
    ```

    * **Note**: Blocking requests wait for the lesser of 60 seconds or the action's configured [time limit](https://github.com/apache/incubator-openwhisk/blob/master/docs/reference.md#per-action-timeout-ms-default-60s).  Actions that take longer will still keep running, but wou will have use the *Activation ID* to lookup the results later just like a non-blocking invocation.

##### Non-blocking invocation

A non-blocking invocation will invoke the action immediately, but not wait for a response.  In this case, the *Activation ID* will be used to find the results.

1. Invoke the `hello` action using the command line as a non-blocking activation:

    ```bash
    ibmcloud fn action invoke hello
    ```

    ```bash
    ok: invoked /_/hello with id 6bf1f670ee614a7eb5af3c9fde813043
    ```

2. Retrieve the activation result using the activation ID from the invocation:

    ```bash
    ibmcloud fn activation result 6bf1f670ee614a7eb5af3c9fde81304
    ```

    You should see similar output as when you previously used the `--result` flag on the `invoke` command.

    <details>
    <summary>Sample output:</summary>
    ```json
    {
        "payload": "Hello World!"
    }
    ```
    </details>

3. Retrieve the full activation record. To get the complete activation record use the `activation get` command using the activation ID from the invocation:

    ```bash
    ibmcloud fn activation get 6bf1f670ee614a7eb5af3c9fde813043
    ```

    You should see the complete activation record as when you previously sed the `--blocking` flag on the `invoke` command.

    <details>
    <summary>Sample output:</summary>
    ```json
    ok: got activation 6bf1f670ee614a7eb5af3c9fde813043
    {
      ...
      "response": {
          "result": {
              "payload": "Hello World!"
          },
          "size": 25,
          "status": "success",
          "success": true
      },
      ...
    }
    ```
    </details>

4. Retrieve the last activation record:

    ```bash
    ibmcloud fn activation get --last
    ```

5. Retrieve the msot recent recent activation list:

    ```bash
    ibmcloud fn activation list
    ```

    <details>
    <summary>Sample output:</summary>
    ```bash
    Datetime   Activation ID  Kind      Start Duration Status  Entity
    y:m:d:hm:s 44794bd6...    nodejs:10 cold  34s      success <NAMESPACE>/hello:0.0.1
    y:m:d:hm:s 6bf1f670...    nodejs:10 warm  2ms      success <NAMESPACE>/hello:0.0.1
    ```
    </details>

    Note: the last 'N' cached activations are shown.

### Hello world with parameters

Event parameters can be passed to an action's function when it is invoked. Let's look at a sample action which uses the parameters to calculate the return values.

#### Update action

1. Update the file `hello.js` with the following source code:

    ```javascript
    function main(params) {
        return {payload:  'Hello, ' + params.name + ' from ' + params.place};
    }
    ```

2. Update the `hello` action with the updated source code file:

    ```bash
    ibmcloud fn action update hello hello.js
    ```

#### Invoke using command line parameters

When invoking actions through the command line, parameter values can be explicitly passed using the `—param` flag or the shorter `-p` flag.

1. Invoke the `hello` action using explicit command-line parameters using the `--param` flag:

    ```bash
    ibmcloud fn action invoke --result hello --param name Elrond --param place Rivendell
    ```

    Or you can use the `-p` short form:

    ```bash
    ibmcloud fn action invoke --result hello -p name Elrond -p place Rivendell
    ```

    ```json
    {
        "payload": "Hello, Elrond from Rivendell"
    }
    ```

### Invoke using parameter file

You can also pass parameters from a file containing the desired content in JSON format. The filename must then be passed using the `--param-file` flag.

1. Create a file named `parameters.json` containing the following JSON:

    ```json
    {
        "name": "Frodo",
        "place": "the Shire"
    }
    ```

2. Invoke the `hello` action using parameters from the JSON file:

    ```bash
    ibmcloud fn action invoke hello --param-file parameters.json --result
    ```

    ```json
    {
        "payload": "Hello, Frodo from the Shire"
    }
    ```

### ZIP action

The NodeJS runtime, where your function executes, has a [fixed list of installed NPM modules](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-runtimes#openwhisk_ref_javascript_environments).  If you require more NPM modules, you can ZIP them with your action code.

##### Fantasy name generator

This example shows how to package a fun NPM module called [Fantasy Name Generator](https://www.npmjs.com/package/fantasy-name-generator) with an action function.

1. Create a project directory and change into it:

```bash
mdkir namegen
cd namegen
```

2. Create `namegen.js` with the following code:

```javascript
function namegen(params) {

    const generator = require('fantasy-name-generator');
    const usage = "-p race [see https://www.npmjs.com/package/fantasy-name-generator] -p gender [male|female]"

    if (params === undefined || params.race === undefined ||
        params.gender === undefined || ( params.gender != 'male' && params.gender != 'female' )) {
            return { usage: usage };
        }

    var name = generator.nameByRace(params.race,{gender: params.gender});

    return{ name: name };
}

exports.main = namegen;
```

3. Create `package.json` with these contents:

```javascript
{
  "name": "namegen",
  "version": "1.0.0",
  "description": "Serverless fantasy name generator",
  "main": "namegen.js",
  "dependencies": {
    "fantasy-name-generator": "^2.0.0"
  }
}
```

4. Install NPM required modules locally:

```bash
npm install
```

5. ZIP the project files with local `node_modules`:

```bash
zip -r action.zip *
```

6. Create the action

Since a ZIP action does not have a `.js` extension, we must use the `--kind` parameter to tell ICF what runtime and version to use (`default` to latest in this example):

```bash
ibmcloud fn action update namegen action.zip --kind nodejs:default
```

7. Invoke the action with parameters (block for `-r` result):

```bash
ic fn action invoke namegen -p race human -p gender female -r
{
    "name": "Aldrella"
}

```

### Asynchronous action

JavaScript functions that run asynchronously may need to return the activation result after the `main` function has returned. You can accomplish this by returning a Node.js `Promise` in your action.

1. Save the following content in a file called `asyncAction.js`:

```javascript
function main(args) {
     return new Promise(function(resolve, reject) {
       setTimeout(function() {
         resolve({ done: true });
       }, 2000);
    })
 }
```

Notice that the `main` function returns a promise, which indicates that the activation hasn't completed yet, but is expected to in the future.

2. Create the action and invoke it:

   ```bash
   ibmcloud fn action create asyncAction asyncAction.js
   ```

   ```bash
   ibmcloud fn action invoke --result asyncAction
   ```

   ```json
   {
       "done": true
   }
   ```

   Notice that you performed a blocking invocation of an asynchronous action.

2. Fetch the last activation log to see how long the async activation took to complete:

   ```text
   ibmcloud fn activation get --last
   ```

   ```json
   {
      ...
      "start": 1574133220119,
      "end": 1574133222155,
      "duration": 2036,
      ...
   }
   ```

   Checking the `duration` field in the activation record, you can see that this activation took slightly over two seconds to complete.

### Web action

ICF can turn any action into a `web` accessible action using the `--web` flag on create or update. This allow the function to produce web-ready contents for constructing web sites using configurable Cross-Origin Resource Sharing (CORS) headers.

Web actions are actions that can be called externally using the HTTP protocol from clients like `curl` or web browsers. IBM Cloud Functions (ICF) provides a simple flag, `--web true`, which causes it to automatically create an HTTP accessible URL (endpoint) for any action.

## Create and invoke a web action

Let's turn the `hello` action into a web action!

1. Update the action to set the `--web` flag to `true`:

      ```bash
      ibmcloud fn action update hello --web true
      ```

      ```bash
      ok: updated action hello
      ```

      The `hello` action has now been assigned an HTTP endpoint.

2. Retrieve the web action's URL exposed by the platform for the `hello` action:

      ```bash
      ibmcloud fn action get hello --url
      ```

      ```bash
      ok: got action hello
      https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello

      ```

3. Invoke the web action URL returned using the `curl` command:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello"
      ```

      It looks like nothing happened! In fact, an HTTP response code of `204 No Content` was returned which you can verify if you add the verbose flag `-v`:

      ```bash
      curl -v "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello"
      ```

      ```bash
      ...
      GET /api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello HTTP/2
      Host: us-south.functions.cloud.ibm.com
      User-Agent: curl/7.54.0
      Accept: */*
      * Connection state changed (MAX_CONCURRENT_STREAMS updated)!
      HTTP/2 204
      ...
      ```

{% hint style="info" %}
This unexpected result occurred because you need to tell ICF what `content-type` you expect the function to return since the function did not explicitly set one.
{% endhint %}

4. Invoke the web action URL with a JSON extension using the `curl` command.

     To signal ICF to set the `content-type` to `application/json` on the HTTP response, you need to add `.json` after the action name, at the end of the URL. Try invoking it now:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello.json"
      ```

      You now get a successful HTTP response in JSON format which matches the `.json` extension you added:

      ```json
      {
         "message": "Hello, undefined from Rivendell"
      }
      ```

## Requests with query parameters

Additionally, you can invoke web actions with query parameters.

1. Use curl to invoke the `hello` web action with a `name` query parameter:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello.json?name=Josephine"
      ```

      ```json
      {
         "message": "Hello, Josephine from Rivendell"
      }
      ```

      If you have been following all the exercises in this course, you will see that the `place` parameter has a default value bound to it.

2. Now, try to invoke the `hello` web action with both a `name` and `place` query parameters:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello.json?name=Josephine&place=Austin"
      ```

      ```bash
      {
          "code": "5675a20dbab5e05445d2a55b38236946",
          "error": "Request defines parameters that are not allowed (e.g., reserved properties)."
      }
      ```

      This error is because web actions, by default, finalize (protect) all bound parameters, making them protected from changes on HTTP requests. In this case, the `place` parameter was bound to the value `Rivendell`.

3. Set the `final` annotation to `false`:

      ```bash
      ibmcloud fn action update hello -a final false
      ```

      ```bash
      ok: updated action hello
      ```

      This will override the default protection on bound parameters.

4. Retry the previous invocation with both query parameters:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello.json?name=Josephine&place=Austin"
      ```

      ```bash
      {
          "payload": "Hello, Josephine from Austin"
      }
      ```

<!-- ## Disable web action support (we cannot disable as we use this in API create...)

1. Update the action to set the `--web` flag to `false`:

      ```bash
      ibmcloud fn action update hello --web false
      ```

      ```bash
      ok: updated action hello
      ```

2. Verify the action is no longer externally accessible:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/hello.json"
      ```

      ```json
      {
        "error": "The requested resource does not exist.",
        "code": "1dfc1f7cb457ed4bb1f2978fc75bd31f"
      }
      ``` -->

## Content types and extensions

Web actions invoked through the platform API either have to set the HTTP response header's `content-type` explicitly within the action function or the caller must append a content extension on the URL so the platform will set the `content-type` on the function's behalf.

The platform supports the following content type extensions: `.json`, `.html`, `.http`, `.svg` or `.text` for the request. If no content extension is provided, the platform defaults to `.http`.

In most cases, it is advisable to have the web action set the content type explicitly when possible.

## HTTP request properties

All web actions created using the `--web` flag are also treated as `http` actions, meaning they can be called with different HTTP methods like GET, POST, or DELETE.

HTTP web actions, when invoked, also receive additional HTTP request details as parameters to the action input argument. These include:

1. `__ow_method` \(type: string\): The HTTP method of the request.
2. `__ow_headers` \(type: map string to string\): The request headers.
3. `__ow_path` \(type: string\): The unmatched path of the request \(matching stops after consuming the action extension\).
4. `__ow_body` \(type: string\): The request body entity, as a base64 encoded string when content is binary or JSON object/array, or plain string otherwise. Only present if handling raw HTTP requests, or when the HTTP request entity is not a JSON object or form data.
5. `__ow_query` \(type: string\): The query parameters from the request as an unparsed string. Click [here](https://github.com/apache/openwhisk/blob/master/docs/webactions.md#raw-http-handling) for more information.
6. `__ow_user` \(type: string\): The namespace identifying the ICF authenticated subject. Only present if the `require-whisk-auth` annotation is set. Click [here](https://github.com/apache/openwhisk/blob/master/docs/annotations.md#annotations-specific-to-web-action) for more information.

Web actions otherwise receive query and body parameters as first class properties in the action arguments. Body parameters take precedence over query parameters, which in turn take precedence over action and package parameters.

{% hint style="tip" %}
Web actions can also be [enabled to handle raw HTTP requests](https://cloud.ibm.com/docs/openwhisk?topic=cloud-functions-actions_web#actions_web_raw_enable). This setting allows the function to directly manage the raw HTTP query string and body content, meaning the actions can receive and process `content-types` other than JSON objects.
{% endhint %}

## Control HTTP responses

Web actions can return a JSON object with the following properties to directly control the HTTP response returned to the client:

1. `headers`: A JSON object where the keys are header names and the values are string, number, or boolean values for those headers \(default is no headers\). To send multiple values for a single header, the header's value should be a JSON array of values.
2. `statusCode`: A valid HTTP status code \(default is 200 OK if body is not empty otherwise 204 No Content\).
3. `body`: A string which is either plain text, JSON object or array, or a base64 encoded string for binary data \(default is empty response\).

The `body` is considered empty if it is `null`, the empty string `""`, or undefined.

If a `content-type` header value is not declared in the action result’s `headers`, the body is interpreted as `application/json` for non-string values and `text/html` otherwise. When the `content-type` is defined, the controller will determine if the response is binary data or plain text and decode the string using a base64 decoder as needed. Should the body fail to decode correctly, an error is returned to the caller.

## Additional features of web actions

Web actions have many more features. See the ICF [documentation](https://cloud.ibm.com/docs/openwhisk?topic=cloud-functions-actions_web) for full details on all these capabilities.

### Example - HTTP redirect

If you have a backend URL that keeps changing, you can provide users with a front-end URL backed by a cloud function that performs a seamless redirect.

1. Create a new web action from the following source code in `redirect.js`:

      ```javascript
      function main() {
            return {
                  headers: { location: "https://openwhisk.apache.org/" },
                  statusCode: 302
            };
      }
      ```

      ```bash
      ibmcloud fn action create redirect redirect.js --web true
      ```

      ```bash
      ok: created action redirect
      ```

2. Retrieve the URL for a new web action:

      ```bash
      ibmcloud fn action get redirect --url
      ```

      ```bash
      ok: got action redirect
      https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/redirect
      ```

3. Check that the HTTP response is indeed an HTTP redirect:

      ```bash
      curl -v "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/redirect"
      ```

      ```bash
      ...
      < HTTP/1.1 302 Found
      < X-Backside-Transport: OK OK
      < Connection: Keep-Alive
      < Transfer-Encoding: chunked
      < Server: nginx/1.11.13
      < Date: Fri, 23 Feb 2018 11:23:24 GMT
      < Access-Control-Allow-Origin: *
      < Access-Control-Allow-Methods: OPTIONS, GET, DELETE, POST, PUT, HEAD, PATCH
      < Access-Control-Allow-Headers: Authorization, Content-Type
      < location: https://openwhisk.apache.org/
      ...
      ```

4. Now try the URL in a browser.

      Did your action successfully redirect to the Apache OpenWhisk project website?

## Web actions with non JSON content types

This section includes a few examples of web actions returning other content types. You can follow along by trying these examples in your browser.

### **Example: HTML response**

If you want to dynamically generate HTML content for web browser access, functions are a great way to do so based upon the latest backend data.

1. Create a new web action named `html` from the following source code in html.js:

      ```javascript
      function main() {
         let html = '<html><body><h3><span style="color:red;">Hello World!</span></h3></body></html>'
         return { headers: { "Content-Type": "text/html" },
                  statusCode: 200,
                  body: html };
      }
      ```

      ```bash
      ibmcloud fn action create html html.js --web true
      ```

      ```bash
      ok: created action html
      ```

2. Retrieve the URL for the web action:

      ```bash
      ibmcloud fn action get html --url
      ```

      ```bash
      ok: got action html
      https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/html
      ```

3. Check that the HTTP response is HTML and copy and paste that into your browser:

      ```bash
      curl "https://us-south.functions.cloud.ibm.com/api/v1/web/2ca6a304-a717-4486-ae33-1ba6be11a393/default/html"
      ```

      ```html
      <html><body>Hello World!</body></html>
      ```

### **Example: SVG Response**

You can generate SVG graphics such as statistical or usage graphs based upon live data using a cloud function.

1. Create a new web action named `atom` that has the following code that includes base64-encoded SVG content in the `body`:

      ```javascript
      // The SVG XML image source has been base64 encoded in the "body" param below:
      function main() {
         return { headers: { 'Content-Type': 'image/svg+xml', 'Cache-Control': 'No-Store' },
            statusCode: 200,
            body: `PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9Ii01MiAtNTMgMTAwIDEwMCIgc3Ryb2tlLXdpZHRoPSIyIj4NCiA8ZyBmaWxsPSJub25lIj4NCiAgPGVsbGlwc2Ugc3Ryb2tlPSIjNjY4OTlhIiByeD0iNiIgcnk9IjQ0Ii8+DQogIDxlbGxpcHNlIHN0cm9rZT0iI2UxZDg1ZCIgcng9IjYiIHJ5PSI0NCIgdHJhbnNmb3JtPSJyb3RhdGUoLTY2KSIvPg0KICA8ZWxsaXBzZSBzdHJva2U9IiM4MGEzY2YiIHJ4PSI2IiByeT0iNDQiIHRyYW5zZm9ybT0icm90YXRlKDY2KSIvPg0KICA8Y2lyY2xlICBzdHJva2U9IiM0YjU0MWYiIHI9IjQ0Ii8+DQogPC9nPg0KIDxnIGZpbGw9IiM2Njg5OWEiIHN0cm9rZT0id2hpdGUiPg0KICA8Y2lyY2xlIGZpbGw9IiM4MGEzY2YiIHI9IjEzIi8+DQogIDxjaXJjbGUgY3k9Ii00NCIgcj0iOSIvPg0KICA8Y2lyY2xlIGN4PSItNDAiIGN5PSIxOCIgcj0iOSIvPg0KICA8Y2lyY2xlIGN4PSI0MCIgY3k9IjE4IiByPSI5Ii8+DQogPC9nPg0KPC9zdmc+`
         };
      }
      ```

      ```bash
      ibmcloud fn action create atom atom.js --web true
      ```

      ```bash
      ok: updated action atom
      ```

2. Get the URL for the new atom web action:

      ```bash
      ibmcloud fn action get atom --url
      ```

      ```bash
      ok: got action atom
      https://us-south.functions.cloud.ibm.com/api/v1/web/josephine.watson%40us.ibm.com_ns/default/atom
      ```

3. Copy and paste that URL into your browser to see the image!

      ![atom.svg](images/atom.svg)

*If you want, you can save the [unencoded SVG XML source](images/atom.svg) to your local computer and view it in a text editor.*

### Observations

- **No special code is needed**. You can code with your favorite language!
  - By convention, the `main` function is called. You can always alias "main" to any function in your `.js` file.
- **No build step**. Runtimes for all supported languages are already deployed in ICF server clusters waiting for your function to be deployed and invoked.
- **Node.js runtime inferred**. The Node.js runtime was inferred via the function's `.js` extension. ICF will always use the latest supported Node.js runtime version unless you explicitly set another version with the `--kind` flag.
- **Package dependencies as ZIP files**.  Complex actions can be constructed by packaging required NPM modules within a ZIP file.
- **Promises are supported by default** since ICF invokes functions asynchronously.
- **Creating websites from function output is easy**. You do not need to host a traditional application server to create dynamic content.
- **Secure namespaces**. Your action executes in an IBM Cloud namespace; a default namespace is used if none is supplied. This allows you to apply Identity and Access Management (IAM) control to all actions in a namespace _which is not included in these examples_.