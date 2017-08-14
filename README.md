# Ivee 3D Editor - a smooth and delightful journey using Aurelia and ThreeJS

[Ivee 3d Editor](http://editor.ivee.tech) is an online editor which allows the creation of beautiful 3D presentations. It provides features like adding various 3D objects - cubes, spheres, torus, etc., configuring materials, textures, shaders, providing animations, tweening etc.

![Ivee 3D Editor](http://editor.ivee.tech/svc/data/ivusers/demo@ivee.tech/tNeNGumvJj/images/iv-3d-1.png "Ivee 3D Editor")

In the following article I'm going to explain a bit about the internals of the editor and my journey on building Ivee using Aurelia and ThreeJS.

On the server-side, Ivee 3D Editor uses .NET Web API with a SQL Server database used to store user authentication information. The editor output is stored as JSON file on the server, in a dedicated user space. The database and web application is hosted on Microsoft Azure.

## Why Aurelia?

I started working on Ivee on Dec 2016. It was a pet project, and I couldn't afford to dedicate a lot of time, only at night and during weekends. My regular job is full stack .NET developer and the frontend framework I'm using at work is Angular. That means I didn't have much experience (even knowledge, to be honest) with Aurelia. 

I came across Aurelia by talking to my friend Dragos, who was disappointed by Angular and its awkward syntax - which I never liked it myself. He started exploring Aurelia and I was interested to learn how to use it as well. I thought learning another framework would be beneficial to my technical knowledge. 

At that time, my thought was that if things are not working well with Aurelia, I can always go back to Angular.

## Choosing Aurelia CLI

So I started looking at Rob's video on [aurelia.io](http://aurelia.io) and from the first go I was impressed with the simplicity and naturalness of Aurelia. I installed [Aurelia CLI](https://github.com/aurelia/cli) and in 10 minutes I had a fully running application, with routing, components, data binding and output events. From that point, I had no doubt about my choice, even though I had a little fear that along the way I will encounter difficulties. With this in mind, I set my expectation that my plans to integrate legacy or non-Aurelia libraries might not work as smoothly as I would like. 

Later on, it turned out that this is one of the features that sets Aurelia apart - the ease of integrating almost everything with Aurelia without spending hours and hours of research and bugs chasing. 

## Integrating ThreeJS (and other libraries)

After setting up the app component and the routes, which was a breeze I might say, my next concern was how I could integrate [ThreeJS](https://threejs.org/). I was pretty familar with the library as I tried various JavaScript experiments, but never using TypeScript and Aurelia. After digging a little bit, I found that there was already a [npm package for ThreeJS](https://www.npmjs.com/package/three). I installed it and I checked the Aurelia documentation about how to [configure libraries](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/the-aurelia-cli/10). In no time, I learnt how to modify *aurelia.json* file to include client libraries and I had my little three js scene up and running - the little rotating cube at the XYZ axis that's on the Ivee 3D Editor [home page](http://editor.ivee.tech).

The configuration for three in *aurelia.json* file looks like this:

```json
{
    "name": "three",
    "path": "../node_modules/three/build",
    "main": "three"
},
``` 

The usage in the component is trivial:

```ts
import * as THREE from 'three';

export class Viewer {

    private selectedObj: THREE.Object3D = null;
    protected w: WglUtil = new WglUtil();

    private clock: THREE.Clock = new THREE.Clock();

    private onDocumentMouseDown(e) {
        this.w.dndMouseDown(e);
        this.selectedObj = this.w.dndSelectedObject;
        if (this.selectedObj) {
            let obj: Iv3dObject = <Iv3dObject>this.selectedObj.userData;
            console.log(obj);
        }
    }

    public animate() {
        let elapsedTime = this.clock.getElapsedTime();
        console.log(elapsedTime);
    }

}
```

Other libraries I'm using:

* [OrbitControls](https://www.npmjs.com/package/three-orbitcontrols) - user interaction with the presentation via mouse and touch;
* [dat.GUI](https://www.npmjs.com/package/dat-gui) - a quick and easy to use GUI to configure properties;
* [expr-eval](https://www.npmjs.com/package/expr-eval) - a JavaScript expression evaluator;
* [tween.js](https://www.npmjs.com/package/tween.js) - a tweening engine for simple animations;
* [threeleapcontrols](https://www.npmjs.com/package/threeleapcontrols) - a package for [VR Leap Motion](https://www.leapmotion.com/) integration;
* [three-stereo-effect](https://www.npmjs.com/package/three-stereoeffect) - stereo effect for VR visualization;
* [three-anaglypheffect](https://www.npmjs.com/package/three-anaglypheffect) - effect for 3D Anaglyph visualization;
* [three-asciieffect](https://www.npmjs.com/package/three-asciieffect) - effect for ASCII visualization.


## Authentication

Authentication is a tricky part of single-page applications which use routes. You want to have authenticated pages, like user settings, as well as pages with anonymous access like home page, login, and signup pages. Aurelia helps a lot with its very own **setRoot** method which can be called conditionally:

```ts
    let userSvc: UserService = aurelia.container.get(UserService);
    let url = window.location.href;
    let noAuthPages: string[] = ['home', 'editor', 'viewer', 'login', 'signup'];
    let isNoAuth = noAuthPages.filter(item => url.indexOf(item) >= 0).length > 0;
    aurelia.start().then(() => {
        if (userSvc.isAuthenticated || isNoAuth) {
            aurelia.setRoot('app');
        }
        else {
            aurelia.setRoot('./components/pages/login-page/login-page');
        }
    });
```

On the server side I use token authentication using [ASP.NET Identity framework](https://www.asp.net/identity). The token is passed with every authenticated call. On the client side, the token is stored in the browser's local storage, managed by the **UserService** service class.


## Data

Integrating data with Aurelia was really easy and I loved that there are options to either use Fetch API, using *aurelia-fetch-client* or XMLHttpRequest API using *aurelia-http-client*. Preferable is Fetch [HttpClient](http://aurelia.io/hub.html#/doc/api/aurelia/fetch-client/latest/class/HttpClient), however, for compatibility reasons, you can use the *xhr* plugin.

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

Validation, which is a must when working with forms, was very easy to integrate thanks to the *aurelia-validation* plugin.

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

Then use **ValidationController** to apply the rules by calling the **validate** method which returns a promise:

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

Giving the fact that most of the user work is performed on the client side, I wanted to have a proper way to manage state in Ivee 3D Editor. One great pattern for managing application state is [Redux](http://redux.js.org/docs/introduction/). 

Having used Redux in Angular 2, I was familiar with the pattern: actions, reducers, state, stores, effects, etc. However, Angular 2 takes advantage of [ngrx](https://github.com/ngrx) library which does a lot of heavy lifting and returns Observables ready to use by Angular components.

I couldn't find anything similar for Aurelia, however the standard [Redux NPM packages](https://www.npmjs.com/package/redux) work without any issue. I spent a bit of time - one user story, about the whole 2 weeks sprint - to integrate the packages and create my own functionality for loading and saving editor data as JSON files, but it was really worth it.

The Redux configuration in the *aurelia.json* file looks as follows (redux thunk is used for middleware associated with asynchronous calls, like Ajax or Fetch calls):

```json
{
    "name": "redux",
    "path": "../node_modules/redux/dist",
    "main": "redux.min"
},
{
    "name": "redux-thunk",
    "path": "../node_modules/redux-thunk/dist",
    "main": "redux-thunk.min"
}
```

An example for load data [actions](http://redux.js.org/docs/basics/Actions.html):

```ts
export class DataMgrActions {

    static actionTypes = {
        LOAD_DATA: 'LOAD_DATA',
        LOAD_DATA_SUCCESS: 'LOAD_DATA_SUCCESS',
        LOAD_DATA_FAIL: 'LOAD_DATA_FAIL',
    }

    constructor(private dataSvc: DataService,
        private userSvc: UserService
    ) {
    }

    // action creators

    loadData = (fileName: string) => {
        return {
            type: DataMgrActions.actionTypes.LOAD_DATA,
            payload: fileName
        };
    };

    loadDataSuccess = (data: DataModel) => {
        return {
            type: DataMgrActions.actionTypes.LOAD_DATA_SUCCESS,
            payload: data
        };
    };

    loadDataError = (error) => {
        return {
            type: DataMgrActions.actionTypes.LOAD_DATA_FAIL,
            payload: error
        };
    };

    loadDataSvc = (fileName: string) => {
        return (dispatch, getState) => {
            dispatch(this.loadData(fileName));
            return dispatch(() => {
                return this.dataSvc.loadFile(fileName)
                    .then((response: any) => {
                        let result: SvcResponse = <SvcResponse>response;
                        if (result.result) {
                            return dispatch(this.loadDataSuccess(result.data));
                        }
                        else {
                            let error = new Error(result.message);
                            return dispatch(this.loadDataError(error));
                        }
                    })
                    .catch((error) => {
                        return dispatch(this.loadDataError(error))
                    });
            });
        };
    };
}
``` 

The load data [reducers](http://redux.js.org/docs/basics/Reducers.html):

```ts
export class DataMgrState {
    data: DataModel = null;
    isError: boolean = false;
    error: any = null;
    actionState: ActionState = ActionState.none;
}

const initialState: DataMgrState = {
    data: null, isError: false, error: null, actionState: ActionState.none
}

export function dataMgr(state: DataMgrState = initialState, action: IvAction) {
    switch (action.type) {
        case actions.DataMgrActions.actionTypes.LOAD_DATA:
            return <DataMgrState>{
                data: action.payload.data,
                actionState: ActionState.pending
            };
        case actions.DataMgrActions.actionTypes.LOAD_DATA_SUCCESS:
            return <DataMgrState>{
                data: action.payload,
                actionState: ActionState.completed
            };
        case actions.DataMgrActions.actionTypes.LOAD_DATA_FAIL:
            return <DataMgrState>{
                data: null,
                isError: true,
                error: action.payload,
                actionState: ActionState.completed
            };
        defult:
            return state;
    }
}

}
```

I separated my components into container and presentation components. The containers are dealing with the store, despatching actions and handling response, whereas the presentation components simply get data and output events to their parents, following the unbeatable one-way data flow pattern. 

Here is an example of my *editor-page* component which is the parent of the main 3D editor:

```ts
export class EditorPage {

    fileName: string;
    data: DataModel;

    store: Store<DataMgrState> = createStore(dataMgr,
        applyMiddleware(
            thunk // lets us dispatch() functions
        )
    );
    private dataMgrActions: DataMgrActions;

    private actionState: ActionState = ActionState.none;

    constructor(private ea: EventAggregator,
        private dialogService: DialogService,
        private router: Router,
        private dataSvc: DataService,
        private userSvc: UserService
    ) {
        this.dataMgrActions = new DataMgrActions(this.dataSvc, this.userSvc);
    }

    activate(params) {
        this.fileName = params.fn; // from route
    }

    private loadFile() {
        if (this.fileName) {
            this.actionState = ActionState.pending;
            this.store.dispatch(this.dataMgrActions.loadDataSvc(this.fileName)).then(() => {
                let state: DataMgrState = <DataMgrState>this.store.getState();
                this.actionState = state.actionState;
                if (state.isError) {
                    console.log(state.error);
                    this.raiseOnError(state.error);
                }
                else {
                    this.data = state.data;
                }
            });
        }
    }

}
```

This is the corresponding HTML template, where the **data** property (of **DataModel** type) is passed as input to the presentation component **editor**:

```html
<template>

    <require from="../../editor/editor"></require>
    <editor data.bind="data" save-data.call="saveData(data)" changed.call="editorChanged(data)"></editor>

</template>
```

As usual, there are lots of shades of gray, and, depending on how complex your application is, soon you realise that input data and output events in a component structure of more than three levels is hard to follow, debug and maintain. But don't despair, the smart people behind Aurelia thought of that too and they introduced the [Event Aggregator](https://github.com/aurelia/event-aggregator) which is a beautiful implementation of a well-known pattern. Used wisely, the event aggregator is a powerful tool which can make your code really tidy and easy to understand and maintain.

For example, to manage errors in a central location in your app, you can use Redux and that works nicely. However, using the Event Agreggator you can publish a custom error (applicable at every component level) and let a central component (like **App**) to handle it. 

A quick example is how I check if WebGL is enabled:

```ts
if (!WglUtil.detectWebGL()) {
    this.ea.publish(CustomEventNames.APP_ERROR, { message: 'Your browser doesn\'t support WebGL.' });
}
```

In the root component, *app.ts*, I subscribe to my error custom event then I use a simple dismissable popup component to display the error message to the user:

```ts
this.appErrorSubscr = this.eventAggregator.subscribe(CustomEventNames.APP_ERROR, payload => {
    this.showError = true;
    this.errorMessage = payload.message;
});
```

The property **errorMessage** is passed in the template to the **error** component:

```html

<require from="./components/error/error"></require>
<iv-3d-error show-error.bind="showError" error-message.bind="errorMessage"></iv-3d-error>

```

To avoid memory leaks, always dispose the subscription in the **detached** hook:

```ts
detached() {
    this.appErrorSubscr.dispose();
}
```




## Dynamic content

Ivee 3D Editor is able to display HTML content in configurable panels. My intention was to leverage the power of HTML in the presentations created using the editor.
To be honest, I was prepared to leave this feature out, thinking it might be difficult to integrate dynamic content in a component. Aurelia came to rescue though and this is an eye opener of how powerful Aurelia is. Behind its simplicity there are a lot of complex features which developer can reuse and take advantage of.
After checking the documentation and after digging in a few Stack Overflow questions, I found that there is a neat way to override the view strategy of a component. 

This is the code of my **ContentPanel** component which uses the **ViewCompiler** service to create the component view at runtime, based on content provided by a custom model (**AdditionalContent**). If you check the code, there is a binding context hooked into the view, which means that even after the initial creation, the view will be updated if the content changes, as it happens with a normal, static template. And these features are out of the box, without installing any additional plugin, 3rd party library or framework! I love Aurelia, because things just work! 

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

Another win, I wanted to provide the user with a nice HTML editing experience and I looked for a nice HTML editor that I could integrate. I chose [squire-rte](https://github.com/neilj/Squire). It's pointless to say that integrating the Squire editor was as easy as 1-2-3:

![HTML Editor](http://editor.ivee.tech/svc/data/ivusers/demo@ivee.tech/tNeNGumvJj/images/iv-3d-4.png "HTML Editor")

## Exposing Viewer API

I always liked extensibility. The thought of exposing your own object model to the developer is fascinating. With Aurelia and TypeScript / ES6, I found that this is actually possible, providing you have designed a nice component structure and a properly written API. Ivee 3D Editor has two major features: the editor and the viewer. The editor is the tool that allows the users to create presentations, by adding objects, setting properties, creating animation timelines, etc. and the viewer is the tool that puts all the presentations parts together and executes them.

But what if you want to interact with the presentation at runtime? No problem, Aurelia and TypeScript / ES6 work for you. The current instance of the **Viewer** type is available for scripting, using plain JavaScript:

```ts
    private evaluateInitScripts() {
        if (!this.data.script) {
            return;
        }
        try {
            let scriptCode: string = `
var viewer;
${this.data.script.init ? this.data.script.init : ''}
`;
            this.createScript(scriptCode);
        }
        catch (e) {
            console.log('Init script creation failed. Check the error: ', e);
        }
        try {
            this.createScript(this.data.script.update);
        }
        catch (e) {
            console.log('Update script creation failed. Check the error: ', e);
        }
        try {
            let scriptCode: string = `
viewer = this; // this line exposes the viewer for runtime use
${this.data.script.execInit ? this.data.script.execInit : ''}
`;
            if (this.runExecInit) {
                let fn = new Function(scriptCode);
                fn.call(this);
            }
        }
        catch (e) {
            console.log('Init script evaluation failed. Check the error: ', e);
        }
    }
```

Admittedly, exposing the API opens the door to vulnerabilities, but in my case, the **Viewer** only provides access to the presentation runtime objects. Examples of **Viewer** methods and properties that can be used at runtime:

* ```data: DataModel``` - the presentation data object model;
* ```w: WglUtil``` - an instance of the **WglUtil** class, which contains a set of helper functions for ThreeJS;
* ```findObjectById(uuid: string)``` - finds a presentation object by its unique identifier;
* ```findObjectByName(name: string)``` - finds a presentation object by its unique identifier;
* ```find3dObjectById(uuid: string)``` - finds a 3D object by its unique identifier;
* ```find3dObjectByName(name: string)``` - finds a 3D object by its name.


Just as a quick test, I used this feature to create a panoramic image for the Aurelia presentation (this code is executed at runtime, when the presentation is loaded):

```js
function addSkyBox() {
    var cfg = { srcFile: 'http://localhost/Iv3DEditorApi/data/ivusers/demo@ivee.tech/av8Rgdveqz/images/skyboxsun5deg2.png', size: 1024 };
    viewer.w.addSkyBoxFromFile(cfg, viewer.mainGroup);
}


addSkyBox();

```

![Presentation modified at runtime](http://editor.ivee.tech/svc/data/ivusers/demo@ivee.tech/tNeNGumvJj/images/iv-3d-5.png "Presentation modified at runtime")

## Conclusion

The Ivee 3D Editor is still in alpha. It has many features, timelines, tweening, data sources, shaders, but it's still rough and needs a lot of refining.
However, I think that for a one man job outside working hours is a pretty good achievement. All these features would have been much more painful to implement without Aurelia. 

Aurelia is beautiful because it doesn't stay in your way, it guides you to do things then goes on the side, admitting that you need to focus on your business. It helps and it doesn't ask for anything in return, it is your quiet and supportive friend always close to you during your journey, ensuring that you have all the resources necessary to climb the highest peaks.

We live in amazing times, where dreams get closer and closer to reality. Our job now is way much simpler than before - we only need to learn and use these wonderful gems that people like Rob create. And Aurelia is a gem that doesn't have a huge learning curve and the time invested to learn it is returned thousandfold. 
If I had to add my own motto for Aurelia, it would be as simple as this: "Aurelia - let amazing things happen".





