---
title: Loading YAML files
lang: en
layout: article_with_sidebar
updated_at: 2016-12-14 16:25 +0400
identifier: ref_8BPAwaEG
categories:
- Developer docs
order: 100
---

## Introduction

This guide describes how developers can load updated YAML file of your module without running {% link "X-Cart SDK macro" ref_HvrXVNvJ#loading-yaml-file %}.

Sometimes developers cannot use X-Cart SDK, because for some reason they cannot run console commands. They can still accomplish other tasks that are eased by macros except loading YAML file. This guide shows a way to load YAML file without having to uninstall/reinstall the module.

## Table of Contents

*   [Introduction](#introduction)
*   [Table of Contents](#table-of-contents)
*   [Implementation](#implementation)

## Implementation

We need to create simple PHP script as follows: 

```php
<?php

require_once 'top.inc.php';

$path = 'path/to/your/yaml/file.yaml';

\XLite\Core\Database::getInstance()->loadFixturesFromYaml($path);
```

and run it through the browser. Of course `$path` variable must contain an actual path to your YAML file.
