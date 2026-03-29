---
title: "paroli-ext-acl"
icon: material/shield-account
statistics: true
---

# Power levels

!!! info "Development Roadmap"
    This extension is currently tethered to the completion of `paroli-core-*`. When it reaches a stable form, we will begin working on it.

## Overview

We will implement ACLs via RBAC, where roles are just hashes (no special meaning, just identifiers), sessions or identities can be assigned roles, and the room defines actions with:
- a whitelist of roles/users allowed to perform that action
- a blacklist of roles/users forbidden from performing it

This will work by adding a new validation rule which will be (possibly the last element) in the **room validation chain**.
