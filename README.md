# Dala Skills

AI-readable documentation for the Dala Care GraphQL API.

## Purpose

This repository contains skills documentation that teaches AI assistants how to interact directly with the Dala Care healthcare workforce management platform. Rather than using an MCP server wrapper, AI can use these skills to make GraphQL requests directly to the API.

## Contents

- **skills.md** - Comprehensive guide covering authentication, queries, mutations, data types, and common workflows

## Usage

Provide `skills.md` to an AI assistant as context when you need it to interact with the Dala Care API. The document includes:

- Authentication setup (API key in `Authorization` header)
- Core queries (offices, care recipients, caregivers, visits, availability)
- Core mutations (scheduling, updates, cancellations, care plans)
- Reference types and enums
- Common workflow patterns

## API Endpoints

- **Production:** `https://api.prod.dala.care/graphql`
- **Development:** `https://api.dev.dala.care/graphql`
