---
theme: dracula
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
transition: slide-left
title: FagDag - Mastering EF Core 
mdc: true
---

# FagDag - Mastering EF Core

Learning from common issues when using EF Core

<v-click>
<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Let's get going <carbon:arrow-right class="inline"/>
  </span>
</div>
</v-click>
<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/kenneth-fossen" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
class: text-center
---
## Intro

<div>
  <br>
  <a href="https://github.com/kenneth-fossen/ef-core-workshop" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
  <br>
  <a href="https://github.com/kenneth-fossen/ef-core-workshop" target="_blank" alt="GitHub">Repo</a>
</div>


---
transition: fade-out
---

## What is EF Core

# EF Core 5 -> 6 -> 7 -> 8

# Dapper vs EF Core

# 7 Deadly 

# Sources
# Practice / Challenges

---
layout: image-right
transition: fade-out
image: /kenneth.jpg
---

# $ whoami

```hs {1|2-5|6-9|10-12|all}
$ whoami
{
     name: Kenneth Fossen,
     dep: Dragefjellet@Bouvet,
     email: kenneth.fossen@bouvet.no,
     edu: [
         Bachelor i Datatryggleik
         Master i Programvare Utvikling
     ],
     work: [
         Helse Vest IKT Drift & Sikkerhet
     ]
     current: [
          project: { 
              name: CommonLibrary@Spine 
          },
          Software Developer,
          Security Champion,
          #rustaceans
      ],
 }
```
---
transition: fade-out
layout: default
---

# Overview

<Toc maxDepth="1"></Toc>
<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->
---
layout: default
---
# What is EF Core
---
layout: default
---
# EF Core 6 -> 7 -> 8
---
layout: default
---
# Dapper vs EF Core
---
layout: default
---
# 7 Deadly Sins
---
layout: default
---
# Practice / Challenges
