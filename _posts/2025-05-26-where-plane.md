---
title: 'Where is that plane going? A shortcut for plane-spotters'
date: 2025-05-26
permalink: /posts/2025/05/where-is-that-plane-going/
tags:
  - automation
  - shortcuts
  - REST API
  - Docker
  - integration
  - Shiny
---

A friend of mine has a curious habit: whenever she sees a plane in the sky, she immediately checks Flightradar24 to find out where it's headed and where it came from. It turns out (perhaps unsurprisingly for those in the industry) all sorts of things happen in the sky. There are hot-air balloons for tourists, law enforcement aircraft, pilots in training, commercial flights, military planes... and somehow, all of them are tracked on Flightradar24.

I thought it could be fun to build some kind of automation. The idea was simple: just one tap, and it should return the origin and destination of the flight above you. I worked on this for a grand total of three hours yesterday, and ended up with something that works surprisingly well and does exactly what I had in mind:

![Example screenshot](https://github.com/malmriv/malmriv.github.io/blob/master/images/captura_witpg.jpeg?raw=true)

Without further ado, here are the details on how to reproduce this automation.

## Create an iPhone Shortcut
I've uploaded the shortcut, and you can [download it here](https://github.com/malmriv/WhereIsThatPlaneGoing/blob/main/shortcut/%C2%BFDo%CC%81nde%20va%20ese%20avio%CC%81n%3F.shortcut) (works for iOS and macOS). Apple Shortcuts are surprisingly versatile: you can parse JSON, send and receive HTTP requests, and access a wide range of iOS-specific actions. If you're curious, try building it yourself. The shortcut should:

1. Get your current location.
2. Save it to a variable.
3. Make a GET request to your API, once you’ve deployed it (more on that later!). The request should include two headers with your longitude and latitude.
4. Parse the response JSON.
5. Extract the relevant key.
6. Display it as a message.

## Create an API to handle Shortcut requests
Your iPhone Shortcut will send a simple HTTP request with two headers. We need to route this to Flightradar24’s API ([documentation here](https://fr24api.flightradar24.com/docs/getting-started)), which uses a different syntax. The logic to convert your location into a valid FR24 request isn’t easy to implement in a Shortcut, so we’ll deploy a simple API to handle it.

Some considerations:

- As a rule of thumb, you can see an airplane that’s about 4–5 km away, horizontally, as the crow flies. This is a rough estimate, but it works well in practice.
- Assuming the Earth is a sphere with a radius of 6371 km, a 7 km arc spans about 1.098 milliradians, or ~0.0675 degrees.
- Therefore, a square covering 7 km in all directions from the user can be defined by adding/subtracting 0.0675 degrees to the user’s latitude and longitude.

So, the API should receive those headers, compute the bounding box, and send a GET request to FR24’s API accordingly.

I chose to write the API in R, just to prove how versatile it is, and because I like R :)

## Deploy your API
There are many hosting options. I used Render.com for a few simple reasons: it supports Docker natively, and its free plan is perfect for small projects. The only drawback is something called “winding down”—if your instance hasn’t received requests in a while, it takes a few seconds to start up again. That’s fine for me—it’s a hobby project. Paying about 6 euros a month would solve this, but there’s no rush.

## Technical details
In more precise terms, the automation works like this. The [repo is here](https://github.com/malmriv/WhereIsThatPlaneGoing/tree/main) and is public. A high-level overview:

1. The Shortcut is built using Apple’s tools. Download the [template here](https://github.com/malmriv/WhereIsThatPlaneGoing/blob/main/shortcut/%C2%BFDo%CC%81nde%20va%20ese%20avio%CC%81n%3F.shortcut?raw=true).
2. A `GET` request is sent to the API with two custom headers: `Latitud` and `Longitud`, each containing your location in decimal degrees.
3. The API runs in a Docker container hosted on [Render.com](https://render.com). It includes:
   - A `plumber.R` script exposing the API using [Plumber](https://www.rplumber.io/)
   - An `api.R` script with the logic, using [httr2](https://httr2.r-lib.org/) and [jsonlite](https://cran.r-project.org/web/packages/jsonlite/index.html)
   - A `requirements.txt` file listing necessary R packages
   - A `Dockerfile` to install the packages and run the API
   - A `.csv` file to translate ICAO codes into human-readable names (see point 5)
4. The API receives your location, computes a 7 km bounding box, and queries the Flightradar24 API for aircraft nearby (usually just the most visible one). No extra filtering — the first result is usually enough, even near airports.
5. ICAO codes are translated into airport names using a resource like the [OurAirports database](https://ourairports.com/data/).
6. The Shortcut receives the result as a JSON object with a single `"Message"` field, which is shown as a notification.

## How to reproduce
Getting this running is pretty straightforward. Render.com makes it easy — once you connect your forked GitHub repo, it’ll detect the Dockerfile and do most of the setup for you. In short:

- Fork this repository.
- Get your own Flightradar24 API key.
- Create an account on [Render.com](https://render.com).
- Set up a Docker container (just a few clicks, no Docker knowledge required).
- Connect the container to your forked repo.
- Add an environment variable called `MY_API_KEY` with your Flightradar24 API key.
- The repository already includes all required dependencies.
- Install the Shortcut on your iPhone and grant the necessary permissions (location and HTTP requests via your default browser).

That’s it! Happy plane-spotting :)