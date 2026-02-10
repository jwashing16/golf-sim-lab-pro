# Project: Golf Sim Lab Pro

**Status:** 🟢 Active  
**Started:** 2026-02-10  
**Last Updated:** 2026-02-10  
**Location:** `/home/jeffrey/matty/projects/golf-sim-lab/`

## Overview

Affiliate marketing site for golf launch monitors with AI-assisted, data-driven reviews. Differentiator: Aggregates hundreds of real user experiences vs. single reviewer opinions.

## Key Metrics

- **Domain:** golfsimlabpro.com (purchased, DNS pending)
- **Affiliate ID:** golfsimlabpro-20
- **Live URL:** https://golf-sim-lab-pro.pages.dev/
- **Revenue Model:** Amazon Associates commissions
- **Content:** 5 reviews + 1 buying guide live

## Current Tasks

### 🔴 In Progress (Today)
- [ ] **P1: Fix product images** - Amazon URLs blocked, implementing solution

### 📋 Next Up (This Week)
- [ ] **P1: Connect custom domain** - golfsimlabpro.com → Cloudflare Pages
- [ ] **P1: Submit to Google Search Console** - Get indexed
- [ ] **P2: Build research pipeline** - Reddit API collector, SQLite schema
- [ ] **P2: Create transparency page** - "How we use AI" disclosure

### ✅ Completed
- [x] Domain purchased (Namecheap)
- [x] Amazon Associates account approved
- [x] Site built (HTML/CSS, no frameworks)
- [x] Deployed to Cloudflare Pages
- [x] 5 product reviews written
- [x] Buying guide published
- [x] Affiliate disclosure page
- [x] Privacy policy
- [x] Dashboard integration (Golf Technology program)

## Data Architecture (Design Phase)

Document at `docs/ARCHITECTURE.md`
- 4-layer system: Collection → Processing → Generation → Publication
- Radical transparency: AI aggregates, human reviews, clear attribution
- Living content: Updates monthly as new data arrives

## Blockers

- **Agent spawning disabled:** `sessions_spawn` returns "agentId is not allowed (allowed: none)" despite config showing `maxConcurrent: 8`. This forces all work to be single-threaded, significantly slowing development. Cannot parallelize research, writing, and deployment tasks.
  - Impact: MVP timeline extended from 4-6 hours to 8-10+ hours
  - Workaround: Sequential execution only

- **Product images:** Amazon blocks automated fetching
  - Solution: Using placehold.co temporarily, manual screenshots or host locally

## Notes

- MVP site is live and functional
- Next priority: Custom domain + research pipeline
- Ethical differentiation: "200 user experiences analyzed, not 1 opinion"
