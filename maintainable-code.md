# Code Maintainability Guidelines

Maintainability represents how easily a software system can be modified and it is one of the main quality chracteristics.

Below, you can find some of the basic principles that enable better code maintainability. 

## Short Units of Code

Shorter units (methods, contructors) are easier to analyze, test and reuse.

 - 👉 try to keep it up to 15 lines
 - 👉 consider cohesion, reusability and single responsibility

### how

- ✅ extract code to methods
- ✅ replace method with method object

## Simple Units of Code

Units with fewer decision points are easier to understand, change and test.

## Write Code Once

Duplication of source code should be avoided at all times, since changes will need to be made in each copy. Duplication is also a source of regression bugs.

## Keep Unit Interfaces Small

Units (methods, constructors) with fewer parametres are easier to test and reuse.

## Separate Concerns in Modules

Module (classes) thar are loosely coupled are easier to modify and lead to a more modular system.

## Couple Architecture Components Loosely

Top-level components of a ssytem that are more loosely coupled are easier to modify and lead to a more modular system.

## Keep the Codebase Small

A large system is difficult to maintain, because more code needs to be analyzed, changed and tested. 

## Clean Code

Having irrelevant artifacts such as TODOs and dead code in your codebase makes it more difficult for someone who is not familiar with the codebase to understand and become productive.