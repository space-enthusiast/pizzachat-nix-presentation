---
theme: default
title: Solving Dev Environment Bottlenecks with Nix + direnv
info: Pizzachat presentation
highlighter: shiki
transition: slide-left
---

# Solving Dev Environment Bottlenecks
## with Nix + direnv

Pizzachat Presentation

---

# The Problem

- Every project needs different environment settings
- One project's environment can pollute the global environment
- We have to relearn how to set up each project every time
- Dependencies can collide when they share the same paths (DLL Hell)

---

# Problem Example

Project A needs **OpenSSL 1.1**, Project B needs **OpenSSL 3.0**

Using either project breaks the other.

```
/usr/lib/libssl.so → ???
```

---

# The Solution

- **Nix** — reproducible package manager and build system
- **direnv** — automatically loads environment variables when you enter a directory

---

# How It Works

```nix {all|3-5|6-9|all}
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixpkgs-unstable";

  outputs = { nixpkgs, ... }:
    let pkgs = nixpkgs.legacyPackages.x86_64-linux;
    in {
      devShells.default = pkgs.mkShell {
        buildInputs = [ pkgs.python3 pkgs.nodejs ];
      };
    };
}
```

---

# direnv Integration

`.envrc` — just one line:

```bash
use flake
```

Enter the directory → environment loads automatically.

---

# Problem Solved

- Two Python projects using different Python versions
- Two Node.js projects using different Node.js versions
- Two Java projects with different JDKs
- Any combination — each project is isolated

---

# Limitations of Nix

- Frequently updated packages have update latency
- Nix packages and native language packages can desync
- GUI applications don't work smoothly

---

# Working Around the Limitations

- **Nix for the environment**, native package managers for the ecosystem
- GUI solutions — still a work in progress :(

---

# What I Learned

- There are no silver bullets — choose the right tool for the job
- Future plan: full desktop environment with NixOS

---
layout: center
---

# Q & A
