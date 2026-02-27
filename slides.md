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

# About Me

Github [@space-enthusiast](https://github.com/space-enthusiast)

5+ year developer whos passionate about enhancing Developer Experience (DX)

---

# Problem Example

You're working on two projects:

| | Project A | Project B |
|---|---|---|
| Node | 18 LTS | 20 LTS |
| Terraform | 1.5 | 1.7 |
| ffmpeg | 5.1 | |

---

# Problem Example

```bash {all|1-4|6-9|11-14|all}
$ cd project-a
$ nvm use 18
$ tfenv use 1.5
Now using node v18.19.0 / Terraform v1.5.7

$ cd ../project-b
$ nvm use 20
$ tfenv use 1.7
Now using node v20.11.0 / Terraform v1.7.5

$ cd ../project-a
$ node --version && terraform --version
v20.11.0 / Terraform v1.7.5
# Forgot to switch back! Wrong versions for both!
```

---

# The Problem

- Every tool needs its own version manager (`nvm`, `tfenv`, `pyenv`, ...)
- You must remember to switch versions manually every time
- One mistake breaks your project

---

# Problem Example

| | Project A | Project B |
|---|---|---|
| Node | 18 LTS | 20 LTS |
| Terraform | 1.5 | 1.7 |
| ffmpeg | 5.1 | 6.1 (new) |

---

# Problem Example

```bash {all|1-2|4-5|7-9|all}
$ cd project-b
$ sudo apt upgrade ffmpeg  # Upgrades to 6.1 globally

$ ffmpeg -version
ffmpeg version 6.1

$ cd ../project-a
$ ffmpeg -version
ffmpeg version 6.1  # Project A needs 5.1 — broken!
```

---

# Problem Example

And what about **ffmpeg**?

- No version manager exists
- Always uses global install
- Project A needs 5.1, Project B needs 6.1 — too bad!

---

# The Problem

- Managing multiple toolchain versions across projects is hard
- Global environment pollution — one upgrade breaks other projects

---

# We Need

- Project-level isolated environments
- Automatic activation when entering a project

---

# The Solution

- **Nix** — reproducible package manager and build system
- **direnv** — automatically loads environment variables when you enter a directory

---

# Why Nix?

Nix is a package manager that supports the following:

- Isolated environments per project
- Multiple versions of the same tool can coexist
- No global pollution — each project gets exactly what it needs

**Bonus:**
- Declarative configuration
- Reproducible on any machine

---
layout: center
---

# Let's Nixify the project

---

# Nixifying...

| | Project A | Project B |
|---|---|---|
| Node | 18 LTS | 20 LTS |
| Terraform | 1.5 | 1.7 |
| ffmpeg | 5.1 | 6.1 |

---

# Project A: flake.nix

```nix {all|2-4|9-11|all}
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/abc...";      # Node 18
  inputs.nixpkgs-tf.url = "github:NixOS/nixpkgs/def...";   # Terraform 1.5
  inputs.nixpkgs-ff.url = "github:NixOS/nixpkgs/ghi...";   # ffmpeg 5

  outputs = { nixpkgs, nixpkgs-tf, nixpkgs-ff, ... }: {
    devShells.default = mkShell {
      buildInputs = [
        nixpkgs.nodejs_18        # from abc commit
        nixpkgs-tf.terraform     # from def commit
        nixpkgs-ff.ffmpeg_5      # from ghi commit
      ];
    };
  };
}
```

---

# Project B: flake.nix

```nix {all|2-4|9-11|all}
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/xyz...";      # Node 20
  inputs.nixpkgs-tf.url = "github:NixOS/nixpkgs/uvw...";   # Terraform 1.7
  inputs.nixpkgs-ff.url = "github:NixOS/nixpkgs/rst...";   # ffmpeg 6

  outputs = { nixpkgs, nixpkgs-tf, nixpkgs-ff, ... }: {
    devShells.default = mkShell {
      buildInputs = [
        nixpkgs.nodejs_20        # Node 20 (not 18)
        nixpkgs-tf.terraform     # Terraform 1.7 (not 1.5)
        nixpkgs-ff.ffmpeg_6      # ffmpeg 6 (not 5)
      ];
    };
  };
}
```

Different commits → Different versions → Both projects coexist!

---

# Using the Flake

```bash {all|1-4|6-9|all}
$ cd project-a
$ ls
...                       # project files
flake.nix                 # nix config file

$ nix develop
$ node --version && terraform --version && ffmpeg -version
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1
# All tools ready!
```

---

# Using the Flake

```bash {all|2|3-4|6|7-8|all}
$ cd project-a
$ nix develop                # Enter the dev shell from flake.nix
$ node --version && terraform --version && ffmpeg -version
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1

$ cd ../project-b
$ nix develop                # Enter project-b's dev shell
$ node --version && terraform --version && ffmpeg -version
v20.11.0 / Terraform v1.7.5 / ffmpeg 6.1
```

---
layout: center
---

# I don't want to type `nix develop` every time...

---

# Adding direnv

**direnv** — automatically runs commands when you enter a directory

```bash {all|1-5|7-9|all}
$ cd project-a
$ ls
...                       # project files
flake.nix                 # nix config file
.envrc                    # contains: use flake

$ cd .                    # direnv detects .envrc → runs nix develop
$ node --version && terraform --version && ffmpeg -version
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1  # Automatic!
```

---

# Result with direnv

```bash
$ cd project-a
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1  # Automatically loaded!

$ cd ../project-b
v20.11.0 / Terraform v1.7.5 / ffmpeg 6.1  # Switched automatically!
```

No more `nvm use`, `tfenv use`, or `nix develop` — just `cd`!

---
layout: center
---

# Thank you for listening!

---
layout: center
---

# Q & A
