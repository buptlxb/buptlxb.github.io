---
layout: post
title: "Some Common Weakness Enumeration"
description: "a biref introduction to some common weakness enumeration"
category: "Security"
tags: ["Security"]
tagline: ""
---
	Here is a common weakness enumeration. Maybe simple but useful.

## Insecure Interaction Between Components

    These weaknesses are related to insecure ways in which data is sent adn
    received between separate components, modules, programs, procesces, threads
    or systems.

#### Improper Neutralization of Special Elements used in an OS Command ('OS
Command Injection')

The software constructs all or part of an OS command using
externally-influenced input from an upstream component, but it does not
neutralize or incorrectly neutralizes special elements that could modify
the intended OS command when it is sent to a downstream component.

#### Improper Neutralization of Input During Web Page Generation ('Cross-size
Scripting')

The software does not neutralize or incorrectly neutralizes user-controllable
input before it is placed in output that is used as a web page that is served
to other users.

#### Improper Neutralization of Special Elements used in an SQL Command('SQL
Injection')

The software constructs all or part of an SQL command using
externally-influenced input form an upstream component, but it does not
neutralization or incorrectly neutralizes special elements that could modify
the intended SQL command when it sent to downstream component.

#### Cross-Site Request Forgery (CSRF)

The web page does not, or cannot, sufficiently verify whether a well-formed, valid,
constient request was intentionally provided by the user who submitted the
request.

#### Unrestricted Upload of File with Dangerous Type

The software allows attackers to upload or transfer files of dangerous types
that can be automatically processed within the product`s environment.

#### URL Redirected to Untrusted Site ('Open Redirect')

A web page accepts user-controllable input that specifies a link to external
site, and use that link in a Redirect. This simpilifies phishing attacks.

## Risky Resource Management

    These weaknesses are related to ways in which software does not properly
    manage the creation, usage, transfer, or destruction of important system
    resources.

#### Imporper Limitation of Pathname to a Restricted Directory ('Path
Tranversal')

The software uses external input to construct a pathname that is intended to
identify a file or directory that is located underneath a restricted parent
directory, but the software does not properly neutralize special elements
within pathname that can cause the pathname to resolve to a location that is
outside of the restricted directory.

#### Buffer Copy without Checkint Size of Input ('Classic Buffer Overflow')

The program copies an input buffer to an output buffer without verifying that
the size of input buffer is less than the size of output buffer, leading to a
buffer overflow.

#### Incorrect Calculation of Buffer Size

The software does not correctly calculate the size to be used when allocating a
buffer, which could lead to a buffer overflow.

#### Uncontrolled Format String

The software uses externally-controlled strings in print-style functions, which
can lead to buffer overflow or data representation problem.

#### Integer Overflow or Wraparound

The software performs a calculation that can produce an integer overflow or
wraparound, when the logic assumes that the resulting value will always be
larger than the original value. This can introduce other weaknesses when the
calculation is used for resource management or execution control.

#### Download of Code without Integrity Check

The product downloads source code or an executable from a remote location and
executes that code without sufficiently verifying the origin or integrity of
the code.

#### Use of Potentially Dangerous Function

The program invokes a potentially dangerous function that could introduce a
vulnerability if it is used incorrectly, but the function can also be used
safely.

#### Inclusion of Functionality from Untrusted Control Sphere

The software imports, requires, or includes executable functionality (such as a
library) from a source that is outside of the intended control sphere.

{% include JB/setup %}
