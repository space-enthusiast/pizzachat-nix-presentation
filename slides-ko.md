---
theme: default
title: nix + direnv 을 이용한 환경 세팅 지옥 탈출기
info: Pizzachat presentation
highlighter: shiki
transition: slide-left
---

# nix + direnv 을 이용한 환경 세팅 지옥 탈출기

Pizzachat Presentation

---

# About Me

코프링조아 Github [@space-enthusiast](https://github.com/space-enthusiast)

5년차 백엔드 개발자, Developer Experience (DX) 개선에 관심이 많습니다

---

# 문제 예시

두 개의 프로젝트를 작업 중입니다:

| | Project A | Project B |
|---|---|---|
| Node | 18 LTS | 20 LTS |
| Terraform | 1.5 | 1.7 |
| ffmpeg | 5.1 | |

---

# 문제 예시

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
# 버전 전환을 깜빡함! 둘 다 잘못된 버전!
```

---

# 문제점

- 모든 툴마다 버전 매니저가 필요함 (`nvm`, `tfenv`, `pyenv`, ...)
- 매번 수동으로 버전을 전환해야 함
- 한 번의 실수로 프로젝트가 망가짐

---

# 문제 예시

| | Project A | Project B |
|---|---|---|
| Node | 18 LTS | 20 LTS |
| Terraform | 1.5 | 1.7 |
| ffmpeg | 5.1 | 6.1 (new) |

---

# 문제 예시

```bash {all|1-2|4-5|7-9|all}
$ cd project-b
$ sudo apt upgrade ffmpeg  # 전역으로 6.1로 업그레이드

$ ffmpeg -version
ffmpeg version 6.1

$ cd ../project-a
$ ffmpeg -version
ffmpeg version 6.1  # Project A는 5.1이 필요한데 — 망함!
```

---

# 문제 예시

**ffmpeg**은 어떻게 해야 할까?

- 버전 매니저가 없음
- 항상 전역 설치만 가능
- Project A는 5.1, Project B는 6.1이 필요 — 방법이 없음!

---

# 문제점

- 여러 프로젝트에서 다양한 툴체인 버전을 관리하기 어려움
- 전역 환경 오염 — 하나를 업그레이드하면 다른 프로젝트가 망가짐

---

# 우리에게 필요한 것

- 프로젝트별 격리된 환경
- 프로젝트 진입 시 자동 활성화

---

# 해결책

- **Nix** — 재현 가능한 패키지 매니저 및 빌드 시스템
- **direnv** — 디렉토리 진입 시 자동으로 환경 변수를 로드

---

# 왜 Nix인가?

Nix는 다음을 지원하는 패키지 매니저입니다:

- 프로젝트별 격리된 환경
- 같은 툴의 여러 버전이 공존 가능
- 전역 오염 없음 — 각 프로젝트가 필요한 것만 정확히 사용

**보너스:**
- 선언적 설정
- 어떤 머신에서든 재현 가능

---
layout: center
---

# 프로젝트를 Nixify 해봅시다

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
        nixpkgs.nodejs_18        # abc 커밋에서
        nixpkgs-tf.terraform     # def 커밋에서
        nixpkgs-ff.ffmpeg_5      # ghi 커밋에서
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
        nixpkgs.nodejs_20        # Node 20 (18 아님)
        nixpkgs-tf.terraform     # Terraform 1.7 (1.5 아님)
        nixpkgs-ff.ffmpeg_6      # ffmpeg 6 (5 아님)
      ];
    };
  };
}
```

다른 커밋 → 다른 버전 → 두 프로젝트가 공존!

---

# Flake 사용하기

```bash {all|1-4|6|7-9|all}
$ cd project-a
$ ls
...                       # 프로젝트 파일들
flake.nix                 # nix 설정 파일

$ nix develop
$ node --version && terraform --version && ffmpeg -version
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1
# 모든 툴 준비 완료!
```

---

# Flake 사용하기

```bash {all|2|3-4|6|7-8|all}
$ cd project-a
$ nix develop                # flake.nix에서 dev shell 진입
$ node --version && terraform --version && ffmpeg -version
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1

$ cd ../project-b
$ nix develop                # project-b의 dev shell 진입
$ node --version && terraform --version && ffmpeg -version
v20.11.0 / Terraform v1.7.5 / ffmpeg 6.1
```

---
layout: center
---

# 매번 `nix develop` 치기 귀찮은데...

---

# direnv 추가하기

**direnv** — 디렉토리 진입 시 자동으로 명령어 실행

```bash {all|1-5|7-9|all}
$ cd project-a
$ ls
...                       # 프로젝트 파일들
flake.nix                 # nix 설정 파일
.envrc                    # 내용: use flake

$ cd .                    # direnv가 .envrc 감지 → nix develop 실행
$ node --version && terraform --version && ffmpeg -version
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1  # 자동!
```

---

# direnv 적용 결과

```bash
$ cd project-a
v18.19.0 / Terraform v1.5.7 / ffmpeg 5.1  # 자동 로드!

$ cd ../project-b
v20.11.0 / Terraform v1.7.5 / ffmpeg 6.1  # 자동 전환!
```

더 이상 `nvm use`, `tfenv use`, `nix develop` 필요 없음 — 그냥 `cd`만 하면 됨!

---
layout: center
---

# 들어주셔서 감사합니다!

---
layout: center
---

# Q & A
