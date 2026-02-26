# What is this

This is a repository for a presentation at a small conference called pizzachat
The presentation is about solving developer environment setting bottlenecks using nix + direnv

## Problem
- for every project we need different environment settings
- a environment setting for one project can affect the global environment and affect other projects
- because every project environment is different we have to relearn everytime how to setup a project environment for each project
- two environment might collide because there dependecies are located in the same folder structure (this is called DLL hell)

## Problem Example
- Project A needs OpenSSL 1.1, Project B needs OpenSSl 3.0 if you use either of the projects the other project will break

## Solution
- Nix (package manager) : Nix is a reproducible package manager and build system.
- direnv : tool to automatically loads environment variables when you enter a directory.

## Problem Solved
- two python projects using different python versions
- two nodejs projects using different nodejs versions
- two java projects
- etc etc

## Limitations of nix
- frequently updated packages update latency
- nix pkgs repository and native language packages desync issues
- gui applications don't work smoothly at all

## Solutions Of the limitations
- Nix for the environment, native package managers for the ecosystem.
- I have yet to find a smooth solution :(

## What I Learned
- there are no silver bullets we have to choose the right tool for the job
- my current future plan with this approach is making my whole desktop environment in to nix using nixos

## Q & A
- TBD
