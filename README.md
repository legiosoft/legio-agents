# LegioSoft Agents

[![LegioSoft logo](images/dark%20purple%20logo.png)](https://legiosoft.net/)

Reusable `AGENTS.md` instruction sets for AI coding agents, based on LegioSoft architecture patterns.

This repository stores project-agnostic agent guidance by technology stack. The goal is to give AI agents enough structure to make code changes that follow [LegioSoft](https://legiosoft.net/) engineering conventions without exposing private repository names, paths, secrets, or local machine details.

## About LegioSoft Agents

LegioSoft Agents represent reusable software architecture patterns used by LegioSoft teams when building production applications. They are not tied to one private project. Each instruction file captures a durable way to organize code, separate responsibilities, and verify changes for a specific technology stack.

These files are designed for AI-assisted development workflows where agents need clear architectural boundaries before editing code. Learn more about LegioSoft and its software engineering work at [legiosoft.net](https://legiosoft.net/).

## What AGENTS.md Is

`AGENTS.md` is a local instruction file for AI coding agents. It explains how a repository is organized, where different kinds of code belong, which conventions should be preserved, and which commands should be used for verification.

In AI beta workflows, these files help make agent behavior repeatable:

- The agent reads the nearest relevant `AGENTS.md`.
- Instructions apply to the folder containing the file and its child folders.
- More specific nested instructions should override broader instructions.
- The content should describe durable engineering rules, not temporary implementation notes.
- The content must not include credentials, private URLs, private filesystem paths, environment values, or customer-specific context.

## Skill Set

- React: structure and conventions for Vite React TypeScript applications with feature-first organization.
- DotNet: reserved placeholder for .NET service structure and conventions.

## Files

- [react/AGENTS.md](react/AGENTS.md): reusable React frontend guidance.
- [dotnet/AGENTS.md](dotnet/AGENTS.md): empty placeholder for future .NET guidance.

## Authoring Rules

When adding or updating an agent file:

- Explain the folder structure, not one private repository.
- Describe where new code should go and why.
- Keep instructions specific enough to guide edits, but generic enough to reuse across projects.
- Prefer stable architecture rules over taste-based preferences.
- Include verification commands when they are common to the stack.

## Website

For company information, services, and related engineering work, visit [LegioSoft software development](https://legiosoft.net/).
