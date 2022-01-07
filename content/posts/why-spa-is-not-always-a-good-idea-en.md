---
title: "[EN] Why SPA is not always a good idea"
date: 2022-01-07T09:41:32+01:00
draft: true
---

Let's assume that you have:

- A task to develop a platform for selling real estate
- A small team of 3 people
- Complete freedom to choose the technology stack

Despite the good prerequisites (as we thought), we managed to completely "kill" the frontend performance of the first prototype version.
At this point, I would like to review the main reasons for our failure and how we were able to get out of it.

<!--more-->

## Choice of technologies for the frontend

We didn't spend a lot of time thinking about which stack to work with and also didn't think about how to support it in the future, since we're making a prototype. We were interested in flexibility and speed of development. In the conditions of uncertainty, these are the key factors. Our choice was FastAPI for the backend and Vue+Quasar for the frontend.

## Deceptive simplicity

A rich material component library and flexibility of Vue allowed us to quickly try out different ideas. From simple UI/UX experiments to dramatic concept changes. The component approach helped us to focus on specific things and business-related problems instead of diving into tech details. Without much effort, we created entire sets of configurable components for our pages.

## Hidden costs

![meme](/why-spa-is-not-always-a-good-idea-1.jpg)

During prototype development, we overlook many things. This is normal, natural, and not a concern by itself. The issue arises only when we don't put it in the plans. Let's take a little closer look at the hidden "price list" in our particular situation:

1. Every deep concept change leaves its own "scar" on the project. When we move fast, change the approach or fundamentally revise the idea of the project, we rarely think about deep cleaning of the project from irrelevant/legacy functionality.
2. "UI now, technical needs later." We'll look at this in more detail a little later.
3. "Why throw away what we have? We've almost got a finished project!". A familiar stumbling block when the basic concept of the project takes shape, and it's time to think about deeper technical things.

## First problems

It was enough to go to lighthouse to realize that there was an emerging problem that was still being hushed up. The resulting report was screaming about performance and SEO problems. The green "PWA" bar wasn't much of a comfort. Meanwhile, tasks directly related to these issues began to appear on the list. And I assure you, finding out the reasons for low performance is not an easy task for someone with small experience in the frontend. It's the imaginary simplicity, which everyone is talking and writing about. But the most interesting consequences occurred when we tried to enable and understand SSR mode. At this point, we realized that we should have developed with this specific feature in mind. Doing it months later in development is very difficult. The interesting thing is that every Vue-based framework has a different approach to SSR, including Quasar. Our ideology of self-sufficient components collapsed because this mode required working through Vuex. The whole SSR looks like an attempt to solve a situation that shouldn't have happened.

## Critical point

The problems listed above have a cumulative effect. The time required to implement new changes increases with each of them. A rather difficult stage for us was a memory leak on the server side. I will be brief, we have never been able to fix it. We "solved" this problem by load balancing with retries and reloading the containers on the fly. At the moment when the price of implementation and support exceeded all imaginable expectations, all attention shifted to the technical side, we decided to reconsider the original approach.

## Reconsidering ideology

This stage should have come a long time ago.

The first conclusion we've got was that the SPA is a bit different from what we need. We should separate the concept of a website and a web application. Pulling heavy runtime, Virtual DOM and the entire application just to show an article or general real estate information. The lazy loading, which doesn't work as expected, adds more fuel to the fire. The main payload of the application are external libraries (e.g. maps) which are loaded in a single file along with the main application, regardless of whether they are needed on the current page.

A second conclusion is a control over the project. Not many people really understand how their project works under the hood and what their code (and not just theirs) goes through before hitting the browser. Underneath the easy start of a project, there are a lot of complicated things, like a webpack with a ton of configurations.

## Alternative?

We decided to go the simpler and more traditional way. The server-side templates fit our requirements well, given that we don't have a lot of dynamics. With one solution, we take control of what goes on the browser side, get rid of SSR issues, and greatly simplify the principles by which our site works.  

Going to extremes, we chose Go and `html/template` from the standard library because we already had platform modules written with Go, and we were familiar with it. We didn't regret our choice and everything works great.

## Migration process

The migration did not happen all at once. We were running 2 parallel projects, putting more resources into the new one. After the first demo beta, we didn't wait to migrate all the functionality and launched the new version. Despite this, the experience of using the site improved many times over.

## Compromises

Of course, along with this migration, we added complexity. Remember? Everything has a price. Fortunately, these complexities are related specifically to the code itself and have little effect on the final result in the browser.

Despite the separation of the code, our controllers have turned into spaghetti. The main reason was the lack of async/await syntax in Go. In fact, the coroutine system in Go is admirable, but quite verbose in some places. Over time, we began to notice the copy-paste approach more and more often. The pattern system isn't flexible enough to break everything up into small components. And let's agree, Vanilla JS can be verbose even for simple things.

These compromises are not critical, but I wish there was a solution for them. That's why the [`kyoto`](https://github.com/kyoto-framework/kyoto) library was born and a topic for a separate article.

## Summary

I have no doubt that someone will mention our incompetence in the frontend area or the wrong choice of framework. That's fair enough, and yet I'd like to mention a few things:

- Modern JS frameworks are typically presented as something easy to start a project with. But no one mentions how difficult it is to develop, maintain and debug that project.
- The underlying principles of frameworks are mostly the same, so it is fair to assume that the problems are similar.
- Many people (including us) forget about the last word in the SPA acronym. Using technology inappropriately has consequences.

I wish those who have read this to the end to draw their own conclusions from our failure and avoid theirs!
