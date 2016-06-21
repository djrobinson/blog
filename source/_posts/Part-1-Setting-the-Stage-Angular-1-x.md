---
title: 'Part 1: Setting the Stage - Angular 1.x'
date: 2016-05-15 11:29:43
tags:
---

Before jumping into any code, I would like to explore the context that has brought on many of the changes we’re seeing in Angular 2.

## An Intro to Web Components

The term “Web Components” refers to a group of standards that can be used in conjunction to create reusable and interoperable widgets in your web page (meaning self contained pieces of functionality contained within a wrapping element).  The term “Web Components” comes from 4 different official W3C standards:

### Templates
The templates standard describes the use of inert chunks of markup that does not need to be initially rendered or run on your document. This allows developers to build injectable components with their own scripts & styling that won’t be run until they have been added to the document (an example would be displaying a new view in an SPA).
### Custom Elements
Allows developers to define their own HTML tags and elements. It also allows developers to use lifecycle callbacks that track the creation and destruction of a given element.
### Shadow DOM
The shadow DOM is a subtree of DOM elements that allows for the encapsulation of scripts and styles that remains separate from the DOM of the main document. This means that custom styling and variable names aren’t leaked to the rest of the document, which allows for development of DOM elements without any need to worry about naming collisions and style overlap.
### Imports
Allows developers to include other customer HTML elements & other components into their own custom element.

When these four specifications are used in conjunction, we exit a world where scripts and styles can clash when they are not developed by the same person.  It creates compartmentalized environment where each element on an HTML document can be thought of as its own mini-app.  Each component can have self contained styles, layout, and logic that can expose pre-defined interfaces for external users to include within their own application.

## Pre-1.5 Component Conflict

Now that we have an idea ow what Web Components are, let’s take a look at some of the issues with Angular 1.x, and why Angular 2 calls for change. With the shift to more self-contained components, there are a few aspects of Angular’s Directive scope & DOM manipulation capabilities that get called into question.

### Inherited Scopes
Unless you explicitly define isolate scope `scope: { ... }` on your directive, the directive will inherit it’s parents scope by default. This doesn’t mesh with the siloed world of components.
### DOM Manipulation
The Angular `link` option doesn’t lend itself well to the interoperability of components.  By allowing your directive code to listen to & manipulate DOM nodes not contained within the directive, the `link` option implies that a directive will remain tied to external templates.  This makes code reuse difficult, as each template will come with its own challenges.
### Two-way binding
If components should be interoperable and composable, they should also define how their internal properties are updated by the outside world.  Much like the critique of inherited scopes, two-way binding exposes internal variables to the modifications of elements sharing the same scope.

Also, the way pre-1.5 is structured, there is no opportunity to use features like the Import standard, and doesn’t include access to the lifecycle callbacks introduced in the Custom Elements standard. Up next, I’ll take a look at the changes in 1.5 that address a lot of these problems.