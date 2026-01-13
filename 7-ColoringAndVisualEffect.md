# Coloring and Visual Effect

> Written by @linrongbin16, first created at 2026-01-13.

This RFC describes an high-level overview for coloring and visual effect system.

## Introduction

Coloring and visual effect system makes a text editor colorful. Except relying on a [syntax parser](6-Syntax.md), it also contains some components:

1. A syntax parser that recognizes text semantics and parses tokens for most programming langauges and structured text contents.
1. A theme that defines a set of colors for all the parsed tokens.
1. A coloring painter that renders both colors and visual effects onto the text and finally print to canvas/terminal.
