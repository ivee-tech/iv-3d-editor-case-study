# Ivee 3D Editor - a smooth and delightful journey using Aurelia and ThreeJS

[Ivee 3d Editor](http://editor.ivee.tech) is just an editor, a 3D editor. Nothing special with it, it provides features like adding various 3D objects - cubes, spheres, torus, etc., configuring materials, textures, shaders, providing animations, tweening etc.

![Ivee 3D Editor](http://editor.ivee.tech/svc/data/ivusers/demo@ivee.tech/tNeNGumvJj/images/iv-3d-1.png "Ivee 3D Editor")

In the following article, I'm going to explain a bit about the internals of the editor and my journey on building Ivee using Aurelia and ThreeJS.

On the server-side, Ivee 3D Editor uses .NET Web API with a SQL Server database used to store user authentication information. The editor output is stored as JSON file on the server, in a dedicated user space. The database and web application is hosted on Microsoft Azure.

## Why Aurelia?

I started working on Ivee on Dec 2016, as a pet project, dedicating very little time, mostly at night and during weekends. My regular job is full stack .NET developer and the frontend framework I'm using at work is Angular. That means I didnd't have much experience (even knowledge, to be honest) about Aurelia. 

I came across Aurelia by talking to a friend who was disappointed by Angular and its awkward syntax - which I never liked it myself. He started exploring Aurelia and I was interested to learn how to use it as well. I thought learning another framerork would be beneficial to my technical knowledge. 

At that time, my thought was that if things are not working well with Aurelia, I can always go back to Angular.

## Choosing Aurelia CLI

So I started looking at Rob's video on [aurelia.io](http://aurelia.io) and from the first go I was impressed with the simplicity and naturalness of Aurelia. I installed [Aurelia CLI](https://github.com/aurelia/cli) and in 10 minutes I had a fully running application, with routing, components, data binding and output events. From that point, I had no doubt about my choice, even though I had a little fear that along the way I will encounter difficulties. With this in mind, I set my expectation that my plans to integrate legacy or non-Aurelia libraries might not work as smoothly as I would like. 

Later on, it turned out that this is one of the features that sets Aurelia apart - the ease of integrating almost everything with Aurelia without spending hours and hours of research and bugs chasing. 

## Integrating ThreeJS (and other libraries)

After setting up the app component and the routes, which was a breeze I might say, my next concern was how I could integrate [ThreeJS](https://threejs.org/). I was pretty familar with the library as I tried various JavaScipt experiments, but never using TypeScript and Aurelia. After digging a little bit, I found that there was already a [npm package for ThreeJS](https://www.npmjs.com/package/three). I installed it and I checked the Aurelia documentation about how to [configure libraries](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/the-aurelia-cli/10). In no time, I learnt how to modify *aurelia.json* file to include client libraries and I had my little three js scene up and running - the little rotating cube at the XYZ axis that's on the Ivee 3D Editor [home page](http://editor.ivee.tech)   

## Authentication

Authentication

## Data

Integrating data with Aurelia was really easy and I loved that there are options to eitherr use Fetch API, using *aurelia-fetch-client* or XMLHttpRequest API using *aurelia-http-client*. Preferable is Fetch [HttpClient](http://aurelia.io/hub.html#/doc/api/aurelia/fetch-client/latest/class/HttpClient), however, for compatibility reasons, you can use the *xhr* plugin.

[Dependency Injection](http://aurelia.io/hub.html#/doc/article/aurelia/dependency-injection/latest/dependency-injection-basics/1) works nicely with Aurelia and you can easily swap one plugin with another. I started with [*xhr*](https://github.com/aurelia/http-client), but later on I moved to fetch client with minimal changes.

Here is a sample code from my data service:

```ts
@inject(HttpClient, UserService)
export class DataService {


    constructor(private httpClient: HttpClient,
        private userSvc: UserService
    ) {
    }

    loadData(url: string, addToken?: boolean) {
        let headers: Headers = new Headers();
        if (addToken) {
            headers.append('Authorization', `Bearer ${this.userSvc.token}`);
        }
        return this.httpClient.fetch(url, {
            credentials: 'include',
            headers: headers
        })
            .then(response => {
                return this.handleResponse(response);
            });
    }
}
```

## Forms

I use forms in the user authentication (login, signup) and user space management (a feature where user can upload, rename, delete, download files, create, rename, delete folders). The two-way binding for controls is natural in Aurelia and you don't need to do anything special. 

How neat and tidy is this template, when you tell Aurelia to bind the **userName** property to your input and apply validation:

```html
<div class="form-group">
    <label for="user">User name</label>
    <input type="text" name="user" class="form-control" placeholder="Username" value.bind="userName & validate">
</div>
```

Validation, which is a must when working with forms, was so easy to integrate thanks to the *aurelia-validation* plugin.

```ts
aurelia.use
    .plugin('aurelia-validation')
```

The validation rules are simply added to the component early, somewhere in the constructor or **attached** hook. I really like the fluent validation API:

```ts
ValidationRules
    .ensure((m: LoginPage) => m.userName).displayName("User name").required()
    .ensure((m: LoginPage) => m.password).displayName("Password").required()
    .on(this);
```

Then use **ValidationController** to apply the rules by calling the **validate** method which returs a promise:

```ts
this.validationController
    .validate()
    .then(result => {
        if (result.valid) {
            this.loginError = false;
            this.loginErrorMessage = null;
            this.login();
        }
        else {
            this.loginError = true;
            this.loginErrorMessage = 'Validaton error(s):';
            for (let error of this.validationController.errors) {
                this.loginErrorMessage += error.message + ' ';
            }
        }
    })
```

Can't be any cleaner than this.

## Using Redux

Giving the fact that most of the user work is performed on the client side, I wanted to have a proper way to manage state in Ivee 3D Editor. As far as I know, the best way to manage application state is using [Redux](http://redux.js.org/docs/introduction/). 

## Dynamic content

Ivee 3D Editor is able to display HTML content in configurable panels. My intention was to leverage the power of HTML in the presentations created using the editor.
To be honest, I was prepared to leave this feature out, thinking it might be difficult to integrate dynamic content in a component. Aurelia came to rescue though and this is an eye opener of how powerful Aurelia is. Behind its simplicity there are a lot of complex features which developer can reuse and take advantage of.
Looking at the documentation, and a little digging in a few Stack Overflow questions I found that there is a neat way to override the view strategy of a component. 

This is the code of my **ContentPanel** component which uses the **ViewCompiler** service to create the component view at runtime, based on content provided byu a custom model (**AdditionalContent**). If look, there is a binding context hook into the view, which means that even after the initial creation, the view will be updated if the content changes, as it happens with a normal, static template. And these features are out of the box, without installing any additional plugin, 3rd party library or framework!!! I love Aurelia, because things just work! 

```ts
import {inject, noView, ViewCompiler, ViewSlot, Container, ViewResources, bindable} from 'aurelia-framework';

import { AdditionalContent } from '../../models/additional-content';

@noView
@inject(ViewCompiler, ViewSlot, Container, ViewResources)
export class ContentPanel {

    private viewCompiler: ViewCompiler;
    private viewSlot: ViewSlot;
    private container: Container;
    private resources: ViewResources;
    private bindingContext = {
        content: new AdditionalContent()
    };

    @bindable content: AdditionalContent;

    constructor(vc, vs, container, resources) {
        this.viewCompiler = vc;
        this.viewSlot = vs;
        this.container = container;
        this.resources = resources;
    }

    reloadView(content: AdditionalContent) {
        let template = `
<template>
    <div class="panel panel-default" id.bind="content.name" css.bind="content.css" show.bind="content.showFlag">
        <div class="panel-heading">
            <h3 class="panel-title pull-left" inner-text.bind="content.title"></h3>
            <button type="button" class="close" data-dismiss="alert" aria-label="Close" click.trigger="content.hide()"><span aria-hidden="true">&times;</span></button>
            <div class="clearfix"></div>
        </div>
        <div class="panel-body iv-3d-content-panel-body" innerhtml.bind="content.content | sanitizeHTML">
        </div>
    </div>
</template>`;
        content.hide = () => {
            this.content.showFlag = false;
        };
        content.show = () => {
            this.content.showFlag = true;
        };
        this.bindingContext = {
            content: content
        };
        let viewFactory = this.viewCompiler.compile(template, this.resources);
        let view = viewFactory.create(this.container);
        view.bind(this.bindingContext);
        this.viewSlot.add(view);
        this.viewSlot.attached();
    }

    contentChanged(newValue: AdditionalContent) {
        if (newValue) {
            this.reloadView(newValue);
        }
    }

}
```

You can see the dynamic content panels in action in my [Aurelia presentation](http://editor.ivee.tech/#/viewer?d=1&fn=aurelia.json), created using Ivee. Click on the topics on the left and you can see the coresponding panel opening.

## Exposing Viewer API

Exposing Viewer API

## Conclusion


Our job is way much simpler - learn and use these wonderful gems that people like Rob create.
If I had to add my own motto for Arelia, it would be like that: Aurelia - make things happen




