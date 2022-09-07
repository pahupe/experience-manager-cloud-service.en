---
title: Getting Started with SPAs in AEM Using React
description: This article presents a sample SPA application, explains how it is put together, and allows you to get up-and-running with your own SPA quickly using the React framework.
exl-id: 13998526-65e7-4d1b-bd47-452bad3780a2
---
# Getting Started with SPAs in AEM Using React {#getting-started-with-spas-in-aem-using-react}

Single page applications (SPAs) can offer compelling experiences for website users. Developers want to be able to build sites using SPA frameworks and authors want to seamlessly edit content within AEM for a site built using SPA frameworks.

The SPA authoring feature offers a comprehensive solution for supporting SPAs within AEM. This article presents a simplified SPA application on the React framework, explains how it is put together, allowing you to get up-and-running with your own SPA quickly.

>[!NOTE]
>
>This article is based on the React framework. For the corresponding document for the Angular framework see [Getting Started with SPAs in AEM - Angular](getting-started-angular.md).

## Introduction {#introduction}

This article summarizes the basic functioning of a simple SPA and the minimum that you need to know to get yours running.

For more detail on how SPAs work in AEM, see the following documents:

* [SPA Introduction and Walkthrough](introduction.md)
* [SPA Editor Overview](editor-overview.md)
* [SPA Blueprint](blueprint.md)

>[!NOTE]
>
>In order to be able to author content within a SPA, the content must be stored in AEM and be exposed by the content model.
>
>A SPA developed outside of AEM will not be authorable if it does not respect the content model contract.

This document will walk through the structure of a simplified SPA created using the React framework and illustrate how it works so you can apply this understanding to your own SPA.

## Dependencies, Configuration, and Building {#dependencies-configuration-and-building}

In addition to the expected React dependency, the sample SPA can leverage additional libraries to make the creation of the SPA more efficient.

### Dependencies {#dependencies}

The `package.json` file defines the requirements of the overall SPA package. The minimum AEM dependencies for a working SPA are listed here.

```
  "dependencies": {
    "@adobe/aem-react-editable-components": "~1.0.4",
    "@adobe/aem-spa-component-mapping": "~1.0.5",
    "@adobe/aem-spa-page-model-manager": "~1.0.3"
  }
```

Because this example is based on the React framework, there are two React-specific dependencies which are obligatory in the `package.json` file:

```
 react
 react-dom
```

The `aem-clientlib-generator` is leveraged to make the creation of client libraries automatic as part of the build process.

`"aem-clientlib-generator": "^1.4.1",`

Further details about it can be found [on GitHub here](https://github.com/wcm-io-frontend/aem-clientlib-generator).

The `aem-clientlib-generator` is configured in the `clientlib.config.js` file as follows.

```
module.exports = {
    // default working directory (can be changed per 'cwd' in every asset option)
    context: __dirname,

    // path to the clientlib root folder (output)
    clientLibRoot: "./../content/jcr_root/apps/my-react-app/clientlibs",

    libs: {
        name: "my-react-app",
        allowProxy: true,
        categories: ["my-react-app"],
        embed: ["my-react-app.responsivegrid"],
        jsProcessor: ["min:gcc"],
        serializationFormat: "xml",
        assets: {
            js: [
                "dist/**/*.js"
            ],
            css: [
                "dist/**/*.css"
            ]
        }
    }
};
```

### Building {#building}

Actually building the app leverages [Webpack](https://webpack.js.org/) for transpilation in addition to the aem-clientlib-generator for automatic client library creation. Therefore the build command will resemble:

`"build": "webpack && clientlib --verbose"`

Once built, the package can be uploaded to an AEM instance.

### AEM Project Archetype {#aem-project-archetype}

Any AEM project should leverage the [AEM Project Archetype](https://experienceleague.adobe.com/docs/experience-manager-core-components/using/developing/archetype/overview.html), which supports SPA projects using React or Angular and leverages the SPA SDK.

## Application Structure {#application-structure}

Including the dependencies and building your app as described previously will leave you with a working SPA package which you can upload to your AEM instance.

The next section of this document will take you through how a SPA in AEM is structured, the important files which drive the application, and how they work together.

A simplified image component is used as an example, but all components of the application are based on the same concept.

### index.js {#index-js}

The entry point into the SPA is of course the `index.js` file shown here simplified to focus on the important content.

```
import ReactDOM from 'react-dom';
import App from './App';
import { ModelManager, Constants } from "@adobe/aem-spa-page-model-manager";

...

ModelManager.initialize().then((pageModel) => {
ReactDOM.render(
    <App cqChildren={pageModel[Constants.CHILDREN_PROP]} cqItems={pageModel[Constants.ITEMS_PROP]} cqItemsOrder={pageModel[Constants.ITEMS_ORDER_PROP]} cqPath={ModelManager.rootPath} locationPathname={ window.location.pathname }/>
, document.getElementById('page'));

});
```

The main function of `index.js` is to leverage the `ReactDOM.render` function to determine where in the DOM to inject the application.

This is a standard use of this function, not unique to this example app.

#### Static Instantiation {#static-instantiation}

When the component is instantiated statically using the component template (e.g. JSX), the value must be passed from the model to the properties of the component.

### App.js {#app-js}

By rendering the app, `index.js` calls `App.js`, which is shown here in a simplified version to focus on the important content.

```
import {Page, withModel } from '@adobe/aem-react-editable-components';

...

class App extends Page {
...
}

export default withModel(App);
```

`App.js` primarily serves to wrap the root components that compose the app. The entry point of any app is the page.

### Page.js {#page-js}

By rendering the page, `App.js` calls `Page.js` listed here in a simplified version.

```
import {Page, MapTo, withComponentMappingContext } from "@adobe/aem-react-editable-components";

...

class AppPage extends Page {
...
}

MapTo('my-react-app/components/structure/page')(withComponentMappingContext(AppPage));

```

In this example the `AppPage` class extends `Page`, which contains the inner-content methods that can then be used.

The `Page` ingests the JSON representation of the page model and processes the content to wrap/decorate each element of the page. Further details on the `Page` can be found in the document [SPA Blueprint.](blueprint.md)

### Image.js {#image-js}

With the page rendered, the components such as `Image.js` as shown here can be rendered.

```
import React, {Component} from 'react';
import {MapTo} from '@adobe/aem-react-editable-components';

require('./Image.css');

const ImageEditConfig = {

    emptyLabel: 'Image',

    isEmpty: function() {
        return !this.props || !this.props.src || this.props.src.trim().length < 1;
    }
};

class Image extends Component {

    render() {
        return (<img src={this.props.src}>);
    }
}

MapTo('my-react-app/components/content/image')(Image, ImageEditConfig);
```

The central idea of SPAs in AEM is the idea of mapping SPA components to AEM components and updating the component when the content is modified (and vice versa). See the document [SPA Editor Overview](editor-overview.md) for a summary of this communication model.

`MapTo('my-react-app/components/content/image')(Image, ImageEditConfig);`

The `MapTo` method maps the SPA component to the AEM component. It supports the use of a single string or an array of strings.

`ImageEditConfig` is a configuration object that contributes to enabling the authoring capabilities of a component by providing the necessary metadata for the editor to generate placeholders

If there is no content, labels are provided as placeholders to represent the empty content.

#### Dynamically Passed Properties {#dynamically-passed-properties}

The data coming from the model are dynamically passed as properties of the component.

## Exporting Editable Content {#exporting-editable-content}

You can export a component and keep it editable.

```
import React, { Component } from 'react';
import { MapTo } from '@adobe/aem-react-editable-components';

...

const EditConfig = {...}

class PageClass extends Component {...};

...

export default MapTo('my-react-app/react/components/structure/page')(PageClass, EditConfig);
```

The `MapTo` function returns a `Component` which is the result of a composition that extends the provided `PageClass` with the class names and attributes that enable the authoring. This component can be exported to later be instantiated in the markup of your application.

When exported using the `MapTo` or `withModel` functions, the `Page` component, is wrapped with a `ModelProvider` component which provides standard components access to the latest version of the page model or a precise location in that page model.

For more information see the [SPA Blueprint document](blueprint.md).

>[!NOTE]
>
>By default you receive the entire model of the component when using the `withModel` function.

## Sharing Information Between SPA Components {#sharing-information-between-spa-components}

It is regularly necessary for components within a single-page application to share information. There are several recommended ways of doing this, listed as follows in increasing order of complexity.

* **Option 1:** Centralize the logic and broadcast to the necessary components for example by using React Context.
* **Option 2:** Share component states by using a state library such as Redux.
* **Option 3:** Leverage the object hierarchy by customizing and extending the container component.

## Next Steps {#next-steps}

* [Getting Started with SPAs in AEM using Angular](getting-started-angular.md) shows how a basic SPA is built to work with the SPA Editor in AEM using Angular.
* [SPA Editor Overview](editor-overview.md) goes into more depth into the communication model between AEM and the SPA.
* [WKND SPA Project](wknd-tutorial.md) is a step-by step tutorial implementing a simple SPA project in AEM.
* [Dynamic Model to Component Mapping for SPAs](model-to-component-mapping.md) explains the dynamic model to component mapping and how it works within SPAs in AEM.
* [SPA Blueprint](blueprint.md) offers a deep dive into how the SPA SDK for AEM works in case you wish to implement SPAs in AEM for a framework other than React or Angular or simply would like a deeper understanding.