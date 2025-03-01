---
id: faq
title: Frequently Asked Questions
sidebar_label: FAQ
---

## What does single-spa do?
single-spa is a top level router. When a route is active, it downloads and executes the code for that route.

The code for a route is called an "application", and each can (optionally) be in its own git repository, have its own CI process, and be separately deployed.
The applications can all be written in the same framework, or they can be implemented in different frameworks.

## Is there a recommended setup?
We recommend a setup that uses in-browser ES modules + [import maps](#what-are-import-maps) (or [SystemJS](https://github.com/systemjs/systemjs) to polyfill these if you need better browser support).  This setup has several advantages:
1. Common libraries are easy to manage, and are only downloaded once. You can also preload these for a small speed boost as well using the standard preload spec.
1. Sharing code / functions / variables is as easy as `import/export`, just like in a monolithic setup
1. Lazy loading applications is easy, which enables you to speed up initial load times
1. Each application (AKA microservice, AKA ES module) can be independently developed and deployed. Teams are enabled to work at their own speed, experiment (within reason as defined by the organization), QA, and deploy on thier own schedules. This usually also means that release cycles can be decreased to days instead of weeks or months
1. A great developer experience (DX): go to your dev environment and add an import map that points the application's url to your localhost. See "[What is the DX like?](#what-is-the-developer-experience-dx-like)" for more details.

## What is the impact to performance?
When setup in the [recommended way](#is-there-a-recommended-setup), your code performance and bundle size will be nearly identical to a single application that has been code-split. The major differences will be the addition of the single-spa library (and SystemJS if you chose to use it). Other differences mainly come down to the difference between one (webpack / rollup / etc.) code bundle and in-browser ES modules.

## Can I have only one version of (React, Vue, Angular, etc.) loaded?
Using the recommended setup, you setup your import map to download that once. Then, tell each application to _not_ bundle that application code; instead, it will given to you at runtime in the browser. See [webpack’s externals](https://webpack.js.org/configuration/externals/) (other bundlers have similar options) for how to do this.

You do have the option of _not_ excluding those libraries (for example if you want to experiment with a newer version or a different library) but be aware of the effect that will have on user's bundle sizes and application speed.

## What are import maps?
[Import maps](https://github.com/WICG/import-maps) improve the developer experience of in-browser ES modules by allowing you to write something like `import React from "react"` instead of needing to use an absolute or relative URL for your import statement. The same is also true of importing from other single-spa applications, e.g. `import {MyButton} from "styleguide"`. The import-map spec is currently in the process of being accepted as a web standard and at the time of writing has been [implemented in Chrome](https://developers.google.com/web/updates/2019/03/kv-storage#import_maps).

## How can I share application state between applications?
In general, we recommend trying to avoid this — it couples those apps together. If you find yourself doing this frequently between apps, you may want to consider that those separate apps should actually just be one app.

Generally, it’s better to just make an API request for the data that each app needs, even if parts of it have been requested by other apps. In practice, if you’ve designed your application boundaries correctly, there will end up being very little application state that is truly shared — your friends list has different data requirements than your social feed.

However, that doesn’t mean it can’t be done. Here are several ways:
1. Create a shared API request library that can cache requests and their responses. If somone hits an API, and then that API is hit again by another application, it just uses the cache
1. Expose the shared state as an export, and other libraries can import it. Observables (like [RxJS](https://rxjs-dev.firebaseapp.com/)) are useful here since they can stream new values to subscribers
1. Use [custom browser events](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Creating_and_triggering_events#Creating_custom_events) to communicate
1. Use [cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies), [local/session storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), or other similar methods for storing and reading that state. These methods work best with things that don't change often, e.g. logged-in user info.

Please note that this is just talking about sharing application state: sharing functions, components, etc. is as easy as an `export` in one project and an `import` in the other.

## Should I use frontend microservices?
If you’ve ran into some of the headaches a monolithic repo has, then you should really consider it.

In addition, if your organization is setup in a Spotify-type model (e.g. where there are autonomous squads that own full-stack features) then microservices on the frontend will fit very well into your setup.

However, if you’re just starting off and have a small project or a small team, we would recommend you stick with a monolith (i.e. not microservices) until you get to the point that scaling (e.g. organizational scaling, feature scaling, etc.) is getting hard. Don’t worry, we’ll be here to help you migrate when you get there.

## Can I use more than one framework?
Yes. However, it’s something you’ll want to consider hard because it splits your front-end organization into specialities that aren’t compatible (e.g. a React specialist may have problems working in an Angular app), and also causes more code to be shipped to your users.

However, it is great for migrations _away_ from an older or unwanted library, which allows you to slowly rip out the code in the old application and replace it with new code in the new library (see Google results for [the strangler pattern](https://www.google.com/search?q=the+strangler+pattern&oq=the+strangler+pattern)).

It also is a way to allow large organizations to experiment on different libraries without a strong commitment to them.

**Just be conscious of the effect it has on your users and their experience using your app.**

## What is the developer experience (DX) like?
If you're using the [recommended setup](#is-there-a-recommended-setup) for single-spa, you'll simply be able to go to your development website, add an import map that points to your locally-running code, and refresh the page.

There's a [library](https://github.com/joeldenning/import-map-overrides) that you can use, or you can even just do it yourself - you'll note that the source code is pretty simple. The main takeaway is that you can have multiple [import maps](#what-are-import-maps) and the latest one wins - you add an import map that overrides the default URL for an application to point to your localhost.

We're also looking at providing this functionality as part of the [Chrome/Firefox browser extension](https://github.com/CanopyTax/single-spa-inspector).

Finally, this setup also enables you to do overrides _in your production environment_. It obviously should be used with caution, but it does enable a powerful way of debugging problems and validating solutions.

As a point of reference, nearly all developers we've worked with **prefer the developer experience of microservices + single-spa** over a monolithic setup.

## Can each single-spa application have its own git repo?
Yes! You can even give them their own package.json, webpack config, and CI/CD process, using SystemJS to bring them all together in the browser.

## Can single-spa applications be deployed independently?
Yes! See next section about CI/CD.

## What does the CI/CD process look like?
In other words, how do I build and deploy a single-spa application?

With the [recommended setup](#is-there-a-recommended-setup), the process generally flows like this:
1. Bundle your code and upload it to a CDN.
1. Update your dev environment's import map to point to the that new URL. In other words, your import map used to say `"styleguide": "cdn.com/styleguide/v1.js"` and now it should say `"styleguide": "cdn.com/styleguide/v2.js"`

Some options on _how_ to update your import map include:
* Server render your `index.html` with the import map inlined. This does not mean that your DOM elements need to all be server rendered, but just the `<script type="systemjs-importmap>` element. Provide an API that either updates a database table or a file local to the server.
* Have your import map itself on a CDN, and use [import-map-deployer](https://github.com/CanopyTax/import-map-deployer) or similar to update the import map during your CI process. This method has a small impact on performance, but is generally easier to setup if you don't have a server-rendered setup already. (You can also [preload](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) the import map file to help provide a small speed boost). See [example travis.yml](https://github.com/openmrs/openmrs-esm-root-config/blob/master/.travis.yml). Other CI tools work, too.
