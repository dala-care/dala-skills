# Dala Care Skills

Automate your home care operations with AI-powered assistance for Dala Care.

## What is This?

This skill enables AI assistants to help you manage your home care agency through Dala Care. Once installed, you can ask your AI assistant to schedule visits, check caregiver availability, update care plans, and more - all through natural conversation.

## Installation

Install this skill using Claude Code or other compatible AI tools:

```bash
npx skills add https://github.com/dala-care/dala-skills
```

## Setup

The skill reads your Dala Care API key from the `DALA_CARE_API_KEY` environment variable. Set it in your shell or environment before starting your AI assistant. Never paste the key into a chat.

```bash
export DALA_CARE_API_KEY=your-api-key
```

The skill calls `api.us.dala.care` or `api.eu.dala.care` directly, so it works in Claude Code and Cowork. In claude.ai chat those domains are blocked unless an org owner adds them to the network allowlist. If your workspace has the dala.care connector, use that in claude.ai instead.

## Safety

The assistant always shows you the exact change and waits for your confirmation before scheduling, changing or cancelling visits, or editing care plans. It treats care recipient data as protected health information and only fetches what each task needs.

## What You Can Do

With this skill installed, your AI assistant can help you:

- **Schedule & Manage Visits** - Create one-time or recurring visits, modify schedules, and cancel when needed
- **Coordinate Caregivers** - Check availability, view assignments, and find the right caregiver for each visit
- **Track Care Recipients** - Access client profiles, visit history, and care plan details
- **Monitor Operations** - View daily schedules, identify open shifts, and track visit status
- **Update Care Plans** - Modify client information and care plan fields

## Regional Endpoints

- **EU:** `https://api.eu.dala.care/graphql`
- **US:** `https://api.us.dala.care/graphql`
