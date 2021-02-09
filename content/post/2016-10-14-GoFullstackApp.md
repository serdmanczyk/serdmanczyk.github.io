---
date: 2016-10-23T20:00:01-07:00
description: Just in time for the holidays
title: "A fullstack Go + ReactJS App deployed with Docker"
---

<pre>
Disclaimer: this project has changed.
</pre>

Nginx has been removed among other fancy things.

Posts about updates pending...

See latest code for info [Release v2.0 Drop Nginx, convert to Traefik](https://github.com/serdmanczyk/Freyr/releases/tag/v2.0)
<pre>
/Disclaimer
</pre>

## What?

I've written a fullstack app: Go backend, ReactJS web frontend, deployed using Docker-Compose.  

I call it Freyr, obligatory screenshots:

![Screenshot 2016-05-09 23.17.39.png](https://github.com/serdmanczyk/Freyr/raw/v1.0/repo_files/Screenshot%202016-05-09%2023.17.39.png)
![Screenshot 2016-05-09 23.17.48.png](https://github.com/serdmanczyk/Freyr/raw/v1.0/repo_files/Screenshot%202016-05-09%2023.17.48.png)
![Screenshot 2016-05-09 23.18.04.png](https://github.com/serdmanczyk/Freyr/raw/v1.0/repo_files/Screenshot%202016-05-09%2023.18.04.png)


You can read more about the app itself on its [github page](https://github.com/serdmanczyk/Freyr/tree/v1.0).  You can even view the app actually running at [frey.erdmanczyk.com](https://freyr.erdmanczyk.com/latest).  In this post I'll share an overview of the app's design.

It's yet another 'arduino' (in this case a [Particle Core](https://docs.particle.io/datasheets/core-datasheet/)) plant sensor project: meausuring temperature, humidity, light, etc. with sensors and an MCU and then sending it to the cloud.  Unlike many other projects, however, the cloud server is custom built.

## Why?

I cover the 'why' of the app on the github page, but why this post?

If you're looking to write an app in Go, there is hardly a lack of resources for learning.  Be it tutorials, examples, existing libraries, etc. Go has a rich community.

My goal with this post isn't to make a mega-tutorial covering all the how-to's of what I did in this project, but to provide a simple example of pulling those components together into a Thing™.

Rather than an exhaustive overview of how to build this from the ground up, this will instead be a high-level overview of the app, its components, and their purposes.  I'll make some assumptions that the reader understands how the underlying technology works, but if you see something cool and want to know more feel free to comment/e-mail ☺.

## How does this work?

![App layout](/images/deploygoapp/layout.png)

- Deployed via docker-compose
- Nginx gateway
- ReactJS frontend
- Go backend w/ Postgres database
- Google oauth authentication, [JWT](https://jwt.io/) managed sessions, HMAC signed API access

## Deployment

Docker-compose was great because I could script my app environments.  This made spinning up and tearing down the apps on the fly super easy.

Another benefit was the provider I used for this project, [DigitalOcean](https://www.digitalocean.com/), has a [docker-machine driver](https://docs.docker.com/machine/drivers/digital-ocean/).  I created my VM with docker-machine, then could use the same docker client commands to deploy my app in production as I do to test on my local machine.

The repository has four Docker Compose files:

- [docker-compose.yml](https://github.com/serdmanczyk/Freyr/blob/v1.0/docker-compose.yml): Production deployment, pulls all files into the images
- [docker-compose.debug.yml](https://github.com/serdmanczyk/Freyr/blob/v1.0/docker-compose.debug.yml): Rather than copying in files, mounts directories into the container to speed launching docker as well as allow live editing of things such as UI files.
- [docker-compose.integration.yml](https://github.com/serdmanczyk/Freyr/blob/v1.0/docker-compose.integration.yml): Runs integration tests, organized as unit tests that require external resources such as database connections.
- [docker-compose.acceptance.yml](https://github.com/serdmanczyk/Freyr/blob/v1.0/docker-compose.acceptance.yml): Runs full-path acceptance tests, performing API calls and asserting results

I used a [Makefile](https://github.com/serdmanczyk/Freyr/blob/v1.0/Makefile) to simplify some commonly used commands e.g. running tests, building binaries and deploying.

## Nginx

My app doesn't need any load balancing yet.  Rather, at this stage I employ Nginx for other features including:

- Terminating SSL
- [Redirecting HTTP -> HTTPS](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf#L29)
- Serving static files and [handling 304](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf#L44) 'Content Not Modifed' responses
- [Gzipping responses](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf#L12) when applicable
- [Routing '/api/' requests](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf#L49) to the backend.

I'm admittedly not an ops or Nginx configuration expert, so I owe a lot to The Google and Stackoverflow for getting this working.

All nginx config is in the [nginx/conf](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf) directory.

## ReactJS Frontend (+ some D3.js)

I picked ReactJS mainly because of how simple it was to get up and going vs. other frameworks.  I wanted to write Go code, I didn't want to spend my time learning what the perfect frontend framework is.  The less time for me spent on the frontend the better, but I also didn't want to half-ass it...  too badly.

All ReactJS and other static files are packed with Nginx in the [nginx/static](https://github.com/serdmanczyk/Freyr/tree/v1.0/nginx/static) directory.  I used the free [grayscale](https://startbootstrap.com/template-overviews/grayscale/) bootsrap theme for CSS/layout.  The background picture I actually [took myself](https://www.flickr.com/photos/erby/19310748252/in/album-72157655281974332/) on a [hike](https://www.flickr.com/gp/erby/bdoKH4).

I use Webpack to build the app files into a single bundle.js.  My development docker-compose file was handy working on this because I could set webpack to watch changes and rebuild on the fly, and updating my changes in the container.

The web page is a single-page app.  All paths on Nginx not matching [`/api`](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf#L49) or a [valid path in the content directory](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/conf/nginx.conf#L46) serve the [index.html](https://github.com/serdmanczyk/Freyr/blob/v1.0/nginx/static/index.html) file.  This allows users to return to 'pages' within the app courtesy of react-router's functionality.  To demonstrate try going to the [demo](https://freyr.erdmanczyk.com), logging in as the demo user, navigating to a chart, and then copying and pasting the url into a new browser tab.

The app also includes charts, for which I chose [d3.js](https://d3js.org/).  It's easy to make fun of Javascript, but there are those who do make good things with it, and D3.js is one of those things.  It took some magic to get d3 to work with React, because of how React uses a virtual DOM representation, but some code from [react-faux-dom](https://github.com/Olical/react-faux-dom) came to the rescue.

## Go backend

Go powers the backend, the meat of the project, powering:

- Negotiating [oauth](https://github.com/serdmanczyk/Freyr/blob/v1.0/oauth/oauth.go#L66) with [Google](https://github.com/serdmanczyk/Freyr/blob/v1.0/oauth/google_oauth.go#L22)
- Issuing signed [JWT's](https://github.com/serdmanczyk/Freyr/blob/v1.0/token/token.go#L39) to [authenticated users](https://github.com/serdmanczyk/Freyr/blob/v1.0/middleware/authorized.go#L54)
- Authorizing [HMAC signed](https://github.com/serdmanczyk/Freyr/blob/v1.0/models/secret.go#L25) web requests for [API access](https://github.com/serdmanczyk/Freyr/blob/v1.0/middleware/authorized.go#L93)
- Receiving, storing, and serving [sensor readings](https://github.com/serdmanczyk/Freyr/blob/v1.0/routes/readings.go)

This was the part I really cared about, by trade I consider myself a backend developer.

I won't do the intro to 'What Go Is™' that seems obligatory on every blog post.

I will say I have really enjoyed programming in Go.  Having a background in C, then C++, then Python working in Go has been a happy combination of what I've enjoyed between those worlds.  It's got some problems to fix, but it gets a lot of things satisfyingly right.

I could spend a whole couple blog posts talking about how I designed the Go backend, but instead I'll just point at some key design features I chose writing it:

- Using interfaces to model data sources: separating concerns in code, enabling mocking for unit tests, and allowing extensible swapping of data sources.
- No global config or config loading in libraries, only loading configuration in main and passing to components via parameters, which also enables easier unit testing.
- Sticking to the standard lib as much as possible (e.g. [net/http](https://golang.org/pkg/net/http/)), making the code more reusable and less dependent on third party libraries.
- Using [contexts](https://github.com/serdmanczyk/Freyr/blob/v1.0/middleware/authorized.go#L40) in http handlers.  I used the [apollo](https://github.com/cyclopsci/apollo) library for chaining, but need to switch to instead conform to context adaptations in Go 1.7+.

One example of how I used interfaces is with the [models.ReadingStore][models.readingstore] which is implemented by the [DB type][DBType].  When implementing handlers such as [PostReadings][PostReadings] and [GetLatestReadings][GetLatestReadings], the functions accept the interface type rather than DB.  This allowed me to test the handlers using a [mock][readingsmock] and the [httptest](https://golang.org/pkg/net/http/httptest/) package in [readings_test.go](https://github.com/serdmanczyk/Freyr/blob/v1.0/routes/readings_test.go#L26).

Rather than using a framework such as [Gin](https://github.com/gin-gonic/gin), [Echo](https://github.com/labstack/echo), or even [Gorilla](http://www.gorillatoolkit.org/) (which actually plays very well with the stdlib) I chose to stick to the standard library as much as possible for handlers.  I only used [negroni](https://github.com/urfave/negroni) to easily get a couple handy middleware's for free.  I didn't want my project code heavily dependent on a third-party framework, lest I leave the project for a bit and come back to find the library deprecated or no longer compatible with updated language versions.  This also allows easier switching between libraries that also play nice the with standard library.

## Ok, what about the sensors?

I mentioned this app's main purpose was to gather and log sensor data.  I also mentioned some security features.  How do those work together?

I briefly hand-waved at the ability to verify API access via signed HTTP requests.  Once a user is logged in, they can access the [`/api/secret`](https://github.com/serdmanczyk/Freyr/blob/v1.0/main.go#L66) endpoint to get a [one-time-only private secret](https://github.com/serdmanczyk/Freyr/blob/v1.0/routes/secret.go#L11).  This can be used to access the API as their user by [signing the request in a certain way](https://github.com/serdmanczyk/Freyr/blob/v1.0/middleware/authorized.go#L154).  The secret can also be [rotated](https://github.com/serdmanczyk/Freyr/blob/v1.0/routes/secret.go#L45) if needed.  I wrote a [CLI](https://github.com/serdmanczyk/Freyr/tree/v1.0/cmd/surtr) to convenience these operations.

*You may notice I named my app [Freyr](https://en.wikipedia.org/wiki/Freyr) and CLI [surtr](https://en.wikipedia.org/wiki/Surtr).  These are references to the Norse God and jötunn adversary at Ragnarok.*

Currently the MCU I use enables registering a webhook for whenever data is posted using a provided firmware function.  This webhook can send a POST request to a specified endpoint, in this case [`/api/reading`](https://github.com/serdmanczyk/Freyr/blob/v1.0/main.go#L70).  Unfortunately, their current functionality as of the time I was doing this work didn't provide methods for request signing, so the best that could be done is to configure the webhook to include a [signed JWT](https://github.com/serdmanczyk/Freyr/blob/v1.0/token/token.go#L107) and headers with the POST request.  Admittedly not the most secure method, but given the amount of free-time I had(/have) it was a compromise for now that got me going.  I mean, people would look at what I've done already and awe themselves with my apparent amount of free-time.

## Other things worth Mentioning

- [Free certs with LetsEncrypt!](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04)
- Automated testing (because why not) with [Travis CI](https://travis-ci.org/serdmanczyk/Freyr)
- [A+ Go Report](https://goreportcard.com/report/github.com/serdmanczyk/freyr) :)
- [Godoc](https://godoc.org/github.com/serdmanczyk/Freyr) compliant

## In Summary

That's a high level overview of the app, with a little more exposition on the Go pieces.  Hopefully this serves as a handy example to others working on similar projects.  In the future I look to add more features to this project, such as the ability to water plants and splitting the backend into smaller microservices.  All in time. 


[models.readingstore]: https://github.com/serdmanczyk/Freyr/blob/v1.0/models/readings.go#L17
[DBtype]: https://github.com/serdmanczyk/Freyr/blob/v1.0/database/reading.go#L9
[PostReadings]: https://github.com/serdmanczyk/Freyr/blob/v1.0/routes/readings.go#L67
[GetLatestReadings]: https://github.com/serdmanczyk/Freyr/blob/v1.0/routes/readings.go#L93
[readingsmock]: https://github.com/serdmanczyk/Freyr/blob/v1.0/fake/reading.go#L43