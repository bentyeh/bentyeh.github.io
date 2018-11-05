---
title: Projects
layout: default
use_fontawesome: true
---

# Research

## Drug Toxicity Target Identification

Manuscript in progress...

# Experience

### Gilead Research Bioinformatics Intern

As an intern on the Research Bioinformatics team, I used gene coexpression networks and transcription factor motif enrichment to study biological mechanisms of non-alcoholic steatohepatitis (NASH) disease progression.

# Projects

### Modeling and Prediction of Drug Toxicity from Chemical Structure

*CS 221: Fall 2017*

Humans are exposed to many different chemical compounds throughout the life course from sources including food, cleaning products, and drugs. In 2014, the [Tox21 data challenge](https://tripod.nih.gov/tox21/challenge/) was launched by the National Center for Advancing Translational Sciences (NCATS) at the U.S. National Institutes of Health (NIH) to better understand the potential of compounds to disrupt biological pathways in possibly toxic ways. The Tox21 library comprises over 10,000 chemical compounds, and data was generated from biological assays measuring each compound’s effect on nuclear receptor signaling and cellular stress pathways.

We trained a 5-layer neural network, achieving a relatively high accuracy of 0.79 and AUC of 0.78 on the validation set. Additionally, by computing the gradient of prediction probabilities with respect to out input features, we were able to recover insights about structure-activity relationships. Carbonyl groups (e.g, ketones, esters, and carboxylic acids) were most predictive of toxicity, while the presence of many aromatic rings tended to be a negative predictor of toxicity.

**Team:** Joyce Kang, Rifath Rashid

<a href="https://drive.google.com/open?id=1z_RtsuLq-luNN4DpRNiYXzcOCEv4-YNJ" class="btn btn-light">
  <i class="fas fa-file"></i> Poster
</a>

<a href="https://github.com/bentyeh/tox21_cs221" class="btn btn-light">
  <i class="fab fa-github"></i> GitHub
</a>

### BAMFermeneter

*BIOE 123: Winter 2018*

We designed and built a fermenter that could be remotely controlled and monitored. Adjustable settings included heat, stirring velocity, and air flow rate; data measurements included optical density and temperature. The user could manually control the system or provide set points for automatic operation.

The physical fermenter was built out of machined plastic and common electrical components (a detailed list is provided on the project repository). We used an Arduino Micro microcontroller to control the fermenter. A Wemos D1 mini WiFi microcontroller connected to the Arduino Micro was linked to Stanford's network to locally host a webservice accessible at bamfermenter.stanford.edu. (Unfortunately, we cannot leave our microcontrollers and circuits powered 24/7, so the website is not available anymore.)

**Team:** Michael Becich, Augustine Chemparathy

<a href="https://github.com/bentyeh/bioe123_BAMFermenter" class="btn btn-light">
  <i class="fab fa-github"></i> GitHub
</a>