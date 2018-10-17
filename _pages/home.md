---
title: "About Me"
permalink: /
excerpt: "Alois Klink is a third-year Electronic Engineering and Artificial Intelligence student at the University of Southampton."
layout: single
author_profile: true
sitemap: true
modified: 2018-10-17

grades:
 - name: Advanced Machine Learning
   code: COMP6208
   grade: 80
 - name: Computational Finance
   code: COMP6212
   grade: 100
 - name: Computational Biology
   code: COMP3212
   grade: 82
 - name: Computer Vision
   code: COMP3204
   grade: 75
 - name: Advanced Computer Vision
   code: COMP6206
   grade: 87
 - name: Advanced Programming
   code: ELEC1204
   grade: 75
 - name: Intelligent Agents
   code: COMP6203
   grade: 78
---

I am a PhD student in the
[Sustainable Energy Research Group](http://www.energy.soton.ac.uk/) in the
University of Southampton, working with
[Professor AbuBakr S. Bahaj](http://www.energy.soton.ac.uk/people/bahaj/) and
[**nquiring**minds](https://nquiringminds.com/).
the University of Southampton.
My current topic is on using Artificial Intelligence and Computer Vision on
Renewable Energy production from Photovoltaic Solar Panels.

I'm mostly interested in programming, especially
in anything related to Artificial Intelligence, such as Machine Learning or
Computer Vision.

## Experience

### AI Engineer

#### nquiringminds Ltd.

##### Jun 2018 - Current

Some of my responsibilities included:

- Using deep-learning for the classification of signals using a
  software-defined radio.
- Working on a Javascript Node.js NPM database package.
- Creating a front-end data visualization responsible web-application using
  React.
- Creating Bash install scripts.
- Designing and building servers and RESTful APIs using OpenAPI/Swagger.

### Embedded Engineering Intern

#### nquiringminds Ltd.

##### Jun 2017 - Sep 2017

I was working on experimenting with high-speed data processing and storage at
[**nquiring**minds](https://nquiringminds.com/).

My work consisted of benchmarking and testing different processing solutions.

It involved:

- Compiling and optimizing TensorFlow on ARM processors
- Designing and benchmarking fast and space-efficient binary-data storage
- Testing and benchmarking fast signal-processing algorithms

This involved writing code in Python, C++, and SQLite, along with testing SIMD
vectorized instructions, compiler optimizations, compression algorithms,
and file systems (ie F2FS vs ext4).

##### Mar 2018 - Apr 2018

I was doing sysadmin with Linux, writing Bash scripts and Systemd
services for automatically creating networks and accesspoints
over WiFi, Ethernet, 3G, and SMS.

### Engineering Intern

#### Airbus Defence & Space Friedrichshafen

##### Jun 2016 - Sep 2016

I was working on the on-board software for the "FLP Testbench" project: a
testbench for a small, affordable, customizable, and state-of-the-art satellite
platform. My work mainly consisted of porting the on-board software to run on a
newer dual-core computer by adapting the code to use asymmetric multiprocessing.
I also upgraded and tested the code with the in-development version of RTEMS 4.11,
and tested symmetric multiprocessing.

The software was written in C++11 with an object-oriented design. To increase code
reusability, the code was heavily encapsulated and filled with abstract
interfaces and generic templates. [Doxygen] [dox_url] was used to make documentation.

*See [the page on Airbus Experience]( {{ "/experience/airbus" | absolute_url }}) for more information.*

[dox_url]: http://www.stack.nl/~dimitri/doxygen/

## Education

### PhD Engineering and the Environment

#### The University of Southampton

##### Sep 2018 - Sep 2022

I'm working in the
[Sustainable Energy Research Group](http://www.energy.soton.ac.uk/) at the
University of Southampton, working with
[Professor AbuBakr S. Bahaj](http://www.energy.soton.ac.uk/people/bahaj/) and
[**nquiring**minds](https://nquiringminds.com/).

My current topic is on using Artificial Intelligence and Computer Vision on
Renewable Energy production from Photovoltaic Solar Panels.

### MEng Electronic Engineering with Artificial Intelligence

#### The University of Southampton

##### Sep 2014 - Jun 2018

###### First Class Honors

The University of Southampton's
[MEng Electronic Engineering with Artificial Intelligence] [elecEng_url] gave me
an overview of the entire field of Electronics with a focus on
Artificial Intelligence.

For my third-year project, I worked on evaluating and assessing
Iterative-Learning-Control algorithms with Professor Eric Rogers.

For my fourth-year project, I was the project leader for simulating a robotic
arm, with my focus on using Convolutional Neural Networks for Computer Vision
with Dr. Klaus-Peter Zauner.

[elecEng_url]: http://www.ecs.soton.ac.uk/programmes/meng-electronic-engineering-artificial-intelligence

Grades:

<table border="1">
	<tr>
		<td> <b> Final Result </b> </td>
		<td> <b> First Class Honors </b> </td>
	</tr>
	{% for course in page.grades %}
	<tr>
		<td> 
		  <a href="http://www.ecs.soton.ac.uk/module/{{ course.code }}">
		    {{ course.name }}
		  </a>
		</td>
		<td> {{ course.grade }} </td>
	</tr>
	{% endfor %}
</table>

### Machine Learning Workshop

#### JP Morgan, London

##### Jun 2016

This JP Morgan Machine Learning Workshop taught machine learning techniques in
Python.

## Projects

### IBM's Master the Mainframe Contest 2016

##### Nov 2016
<div style="overflow:hidden">
  <div style="float:right; width:calc(5% + 150px); text-align: right; height: auto">
    <div data-iframe-width="150" data-iframe-height="270" data-share-badge-id="5bc08906-b832-44d9-8ee4-897469a3bf53"></div>
    <script type="text/javascript">
      (function() {
        var s = document.createElement('script');
        s.type = 'text/javascript';
        s.async = true;
        s.src = '//cdn.youracclaim.com/assets/utilities/embed.js';
        var o = document.getElementsByTagName('script')[0];
        o.parentNode.insertBefore(s, o);
        })();
    </script>
  </div>
  <div markdown="1" style="float:left; width: calc(95% - 150px); height: auto">
I was a [Part 2 Prize Winner] [mtm2016WoF] for IBM's Master the Mainframe Contest 2016.
In it, I learnt how to perform extensive programming (advanced commands, system setup, and advanced system navigation) and application developing (C, JAVA, COBOL, assembler and REXX) tasks, as well as gaining hands-on experience with multiple operating systems (Linux on z Systems, z/VM, z/OS, z/TPF).
  </div>
</div>

[mtm2016WoF]: http://mtm2016.mybluemix.net/wall_of_fame/wall_of_fame.html

### Ball Robot

##### March 2016

I worked on a web-controlled spherical ball robot. I wrote most of the back-end 
code that it ran on, allowing Javascript on a website to execute Python code on
Raspberry Pi in the ball robot, which could then control motors. I also wrote
Javascript (and the necessary backend) that allowed accelerometer/GPS data to be
shown on the website, along with the motor status.

### IBM's Master the Mainframe Contest 2015

##### Feb 2016

I received an [honorable mention] [mtmWoF] for Part 3 of IBM's Master the Mainframe
Contest 2015. It involved heavy use of z/OS and some JCL, Java, COBOL, and SQL.
I was also a [prize winner] [mtmWoF] for Part 2.

[mtmWoF]: http://mtm2015.mybluemix.net/#tabs-6

### PID Controlled Voltage Boost Converter

##### May 2015

I created a boost converter controlled by an [ATmega644P] [atmega]. It was programmed
in C and used a PID controller. In addition, the embedded device communicated to
a PC via UART, where a GUI (also programmed in C) showed statistics and allowed
changing parameters.

[atmega]: http://www.atmel.com/devices/atmega644p.aspx

### Password Manager

##### May 2014

I built a password manager in Java. It had a cryptographically secure random
password generator, and saved the passwords in an encrypted format. The GUI was 
made using Swing. 

## Skills

### Programming Languages

* C (Embedded Programming)
* C++ (Qt and RTEMS)
* MATLAB
* Java
* JavaScript (React and Node.js)
* Python (Numpy and TensorFlow)
* SystemVerilog (Hardware Description Language)

### Software

* Windows
* Linux and Bash
* Git and SVN
* Eclipse and NetBeans
* Doxygen, Javadoc, JSDocs
