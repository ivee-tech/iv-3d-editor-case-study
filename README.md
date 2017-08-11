# Ivee 3D Editor - a smooth and delightful journey using Aurelia and ThreeJS

[Ivee 3d Editor](http://editor.ivee.tech) is just an editor, a 3D editor. Nothing special with it, it provides features like adding various 3D objects - cubes, spheres, torus, etc., configuring materials, textures, shaders, providing animations, tweening etc.

![Ivee 3D Editor](http://editor.ivee.tech/svc/data/ivusers/demo@ivee.tech/tNeNGumvJj/images/iv-3d-1.png "Ivee 3D Editor")

In the following article, I'm going to explain a bit about the internals of the editor and my journey on building Ivee using Aurelia and ThreeJS.

On the server-side, Ivee 3D Editor uses .NET web api with a SQL Server database used to store user authentication information. The editor output is stored as JSON file on the server, in a dedicated user space. The database and web application is hosted on Azure.

## Why Aurelia?

I started working on Ivee on Dec 2016, as a pet project, dedicating very little time, mostly at night and during weekends. My regular job is full stack .NET developer and the frontend framework I'm using at work is Angular. That means I didnd't have much experience (even knowledge, to be honest) about Aurelia. 

I came across Aurelia by talking to a friend who was disappointed by Angular and its awkward syntax - which I never liked it myself. He started exploring Aurelia and I was interested to learn how to use it as well. I thought learning another framerork would be beneficial to my technical knowledge. 

At that time, my thought was that if things are not working well with Aurelia, I can always go back to Angular.

## Choosing Aurelia CLI

So I started looking at Rob's video on [aurelia.io](http://aurelia.io) and from the first go I was impressed with the simplicity and naturalness of Aurelia. I installed Aurelia CLI and in 10 minutes I had a fully running application, with routing, components, data binding and output events. From that point, I had no doubt about my choice, even though I had a little fear that along the way I will encounter difficulties. With this in mind, I set my expectation that my plans to integrate legacy or non-Aurelia libraries might not work as smoothly as I would like. 

Later on, it turned out that this is one of the features that sets Aurelia apart - the ease of integrating almost everything with Aurelia without spending hours and hours of research and bugs chasing. 

## Integrating ThreeJS (and other libraries)

After setting up the app component and the routes, which was a breeze I might say, my next concern was how I could integrate ThreeJS. I was pretty familar with the library as I tried various JavaScipt experiments, but never using TypeScript and Aurelia. After digging a little bit, I found that there was already a [npm package for ThreeJS](https://www.npmjs.com/package/three). I installed it and I checked the Aurelia documentation about how to [configure libraries](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/the-aurelia-cli/10). In no time, I learnt how to modify *aurelia.json* file to include client libraries and I had my little three js scene up and running - the little rotating cube at the XYZ axis that's on the Ivee 3D Editor [home page](http://editor.ivee.tech)   

## Authentication

Authentication

## Data

Integrating data with Aurelia was really easy and I loved that there are options to eitherr use Fetch API, using *aurelia-fetch-client* or XMLHttpRequest API using *aurelia-http-client*. Preferable is Fetch [HttpClient](http://aurelia.io/hub.html#/doc/api/aurelia/fetch-client/latest/class/HttpClient), however, for compatibility reasons, you can use the *xhr* plugin.

[Dependency Injection](http://aurelia.io/hub.html#/doc/article/aurelia/dependency-injection/latest/dependency-injection-basics/1) works nicely with Aurelia and you can easily swap one plugin with another. I started with [*xhr*](https://github.com/aurelia/http-client), but later on I moved to fect client without minimal changes.

Here is a sample code form my data service:

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

Forms

## Using Redux

Using Redux

## Dynamic content

Dynamic content

## Exposing Viewer API

Exposing Viewer API

## Conclusion

Our job is way much simpler - learn and use these wonderful gems that people like Rob create.

```js

var a = 0;

```


