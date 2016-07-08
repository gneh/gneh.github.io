---
layout:            post
title:             "Python Sample"
date:              2016-07-07 08:00:00 +0800
tags:              Language::Python
category:          Language::Python
---

## Traverse Folder
    import os;
    rootDir = './raw_data/uninstall/';
    list_dirs = os.walk(rootDir);
    for root, dirs, files in list_dirs :
        for f in files :
            print f;
            
