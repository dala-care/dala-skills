# Dala Skills

AI-readable documentation for the Dala Care GraphQL API.

## Purpose

This repository contains skills documentation that teaches AI assistants how to interact directly with the Dala Care healthcare workforce management platform. Rather than using an MCP server wrapper, AI can use these skills to make GraphQL requests directly to the API.

## Installation

Install this skill using Claude Code or other compatible AI tools:

```bash
npx skills add https://github.com/careverk/dala-skills
```

## Contents

- **SKILL.md** - Comprehensive guide covering authentication, queries, mutations, data types, and common workflows

## Usage

Once installed, the skill provides context to AI assistants when interacting with the Dala Care API. The document includes:

- Authentication setup (API key in `Authorization` header)
- Core queries (offices, care recipients, caregivers, visits, availability)
- Core mutations (scheduling, updates, cancellations, care plans)
- Reference types and enums
- Common workflow patterns

## API Endpoints

- **Production:** `https://api.prod.dala.care/graphql`
- **Development:** `https://api.dev.dala.care/graphql`
