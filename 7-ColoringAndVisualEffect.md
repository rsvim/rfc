# Coloring and Visual Effect

> Written by @linrongbin16, first created at 2026-01-13.

This RFC describes an high-level overview for coloring and visual effect system.

## Introduction

Coloring and visual effect system makes a text editor colorful. Except relying on a [syntax parser](6-Syntax.md), it also contains some components:

2. A theme that defines a set of colors for all the parsed tokens.
3. A coloring painter that renders colors onto the text and finally print to canvas/terminal.
