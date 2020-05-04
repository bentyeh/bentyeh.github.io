---
layout: projects
title: BAMFermenter: Web-controlled E. coli fermenter
mentors: Dr. Ross Venook, Dr. Kwabena Boahen
setting: Stanford BIOE 123, Winter 2018
team: Michael Becich, Augustine Chemparathy
github: https://github.com/bentyeh/bioe123_BAMFermenter
category: projects
description: We designed and built a fermenter that could be remotely controlled and monitored. Adjustable settings included heat, stirring velocity, and air flow rate; data measurements included optical density and temperature. The user could manually control the system or provide set points for automatic operation.

The physical fermenter was built out of machined plastic and common electrical components (a detailed list is provided on the project repository). We used an Arduino Micro microcontroller to control the fermenter. A Wemos D1 mini WiFi microcontroller connected to the Arduino Micro was linked to Stanford's network to locally host a webservice accessible at bamfermenter.stanford.edu. (Unfortunately, we cannot leave our microcontrollers and circuits powered 24/7, so the website is not available anymore.)
---