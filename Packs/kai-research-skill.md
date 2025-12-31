---
name: Kai Research Skill
pack-id: danielmiessler-kai-research-skill-core-v1.0.0
version: 1.0.0
author: danielmiessler
description: Hybrid research routing that saves Claude tokens by directing queries to Gemini CLI or Perplexity API - simple queries go direct, complex queries spawn research agents
type: skill
purpose-type: [productivity, research, automation]
platform: claude-code
dependencies:
  - danielmiessler-kai-core-install-core-v1.0.0
keywords: [research, gemini, perplexity, tokens, savings, routing, web, search, lookup, explain, compare, current, news]
---

<p align="center">
  <img src="icons/kai-research-skill.png" alt="Kai Research Skill" width="256">
</p>

# Kai Research Skill

> Hybrid research routing that saves Claude tokens by directing queries to external tools - Gemini CLI for explanations, Perplexity API for current info

## Installation Prompt

You are receiving a PAI Pack - a modular upgrade for AI agent systems.

**What is PAI?** See: [PAI Project Overview](../README.md#what-is-pai)

**What is a Pack?** See: [Pack System](../README.md#the-journey-pai-v1x--v20)

This pack adds intelligent research routing to your PAI infrastructure. Instead of Claude handling all research queries (using your tokens), the Research Skill routes queries to external tools:

- **Gemini CLI** - Free, fast explanations with 2M context window
- **Perplexity API** - Real-time web search with citations

**The key insight:** Not all research needs Claude's reasoning. "What is GraphQL?" can go to Gemini (free). "Latest React 19 features" can go to Perplexity (current). Complex queries still spawn research agents for intelligent synthesis.

**Token savings:** 50-63% depending on query type.

Please follow the installation instructions below to integrate this pack into your infrastructure.

---

## What's Included

| Component | File | Purpose |
|-----------|------|---------|
| Main Skill | `Skills/Research/SKILL.md` | Routing patterns and triggers |
| Query Router | `Skills/Research/Tools/QueryRouter.ts` | Analyzes and routes queries |
| Gemini Client | `Skills/Research/Tools/GeminiQuery.ts` | Wraps Gemini CLI |
| Perplexity Client | `Skills/Research/Tools/PerplexityQuery.ts` | Calls Perplexity API |
| Routing Rules | `Skills/Research/Data/RoutingRules.yaml` | Simple vs complex patterns |
| Tool Routing | `Skills/Research/Data/ToolRouting.yaml` | Gemini vs Perplexity patterns |
| Quick Lookup | `Skills/Research/Workflows/QuickLookup.md` | Direct tool call path |
| Deep Research | `Skills/Research/Workflows/DeepResearch.md` | Agent escalation path |

**Summary:**
- **Files created:** 8
- **Skills registered:** 1 (Research)
- **Dependencies:** kai-core-install (required), kai-agents-skill (optional for escalation)

---

## The Concept and/or Problem

Claude Code has built-in WebSearch and WebFetch tools. They work fine. But every research query uses your Claude tokens:

**Current State (No Pack):**
```
You: "What is GraphQL?"
Claude: Uses ~1100 tokens to search, read, and explain
```

**The Problem:**
- Simple explanations don't need Claude's reasoning
- Current info queries hit Claude's knowledge cutoff
- You pay tokens for research that external tools do better
- No way to leverage free tools like Gemini CLI

**The Cost:**
| Query Type | Claude Tokens | Monthly (100 queries) |
|------------|---------------|----------------------|
| Simple lookups | ~1100 each | ~110,000 tokens |
| Current info | ~1100 each | ~110,000 tokens |
| Complex research | ~3000 each | ~300,000 tokens |

**What if:**
- "What is X?" → Gemini (free, 2M context)
- "Latest Y news" → Perplexity (real-time, citations)
- "Research X and recommend for my project" → Agent (intelligent synthesis)

---

## The Solution

The Research Skill implements **hybrid routing** - simple queries go directly to external tools, complex queries spawn research agents.

### Architecture

```
Query arrives
     │
     ▼
┌─────────────────────────────────────┐
│  QueryRouter.ts                      │
│  Analyzes: complexity + topic        │
└─────────────────────────────────────┘
     │
     ├── Simple? ─────────────────────▶ Direct tool call
     │   Pattern: "what is", "explain"     │
     │   No personal context               ▼
     │                              ┌──────────────┐
     │                              │ ToolRouting  │
     │                              │ Gemini vs    │
     │                              │ Perplexity   │
     │                              └──────────────┘
     │                                     │
     │                                     ▼
     │                              Result returned
     │
     └── Complex? ────────────────────▶ Spawn research agent
         Pattern: "research...and recommend"    │
         Personal context: "for my project"     ▼
                                        ┌──────────────┐
                                        │ AgentFactory │
                                        │ (if Agents   │
                                        │ skill exists)│
                                        └──────────────┘
                                               │
                                               ▼
                                        Agent uses both tools
                                        Synthesizes answer
```

### Routing Logic

**Simple → Direct Call (bypass Claude reasoning):**

| Pattern | Tool | Why |
|---------|------|-----|
| "what is", "what are", "define" | Gemini | Conceptual, no current data needed |
| "explain", "describe", "how does...work" | Gemini | Technical depth, large context |
| "latest", "recent", "current", "2025" | Perplexity | Needs real-time web data |
| "news", "announcement", "release" | Perplexity | Current events |
| "compare X vs Y" (no personal context) | Perplexity | Needs current benchmarks/opinions |

**Complex → Agent Escalation (need reasoning):**

| Signal | Example |
|--------|---------|
| Personal context | "for my project", "in my case" |
| Synthesis request | "and recommend", "and decide" |
| Multi-part | "investigate...then analyze...then suggest" |
| Long query | >25 words with multiple requirements |

---

## What Makes This Different

This sounds similar to just using ChatGPT or Gemini directly for research. What makes this approach different?

The Research Skill provides **automatic, in-context routing** integrated directly into your Claude Code workflow. No context-switching between tools. Queries are analyzed and routed transparently - you keep working, external tools handle what they're best at. Results flow back into Claude's context for synthesis. Rules are YAML-configurable. Token savings are measurable.

- Automatic routing prevents manual context-switching overhead
- Results stay in Claude context for seamless continuation
- YAML rules make routing logic transparent and tweakable
- Complex queries still get full agent reasoning treatment

---

## Installation

### Prerequisites

- **Bun runtime**: `curl -fsSL https://bun.sh/install | bash`
- **Gemini CLI**: Must be installed and authenticated
- **Perplexity API key**: Get from [perplexity.ai](https://perplexity.ai)
- **kai-core-install** pack installed
- **Optional**: kai-agents-skill pack (for complex query escalation)

---

### Pre-Installation: System Analysis

```bash
# 1. Check PAI_DIR
echo "PAI_DIR: ${PAI_DIR:-'NOT SET - will use ~/.config/pai'}"

# 2. Check for Gemini CLI
which gemini && gemini --version || echo "WARNING: Gemini CLI not installed"

# 3. Check dependencies
PAI_CHECK="${PAI_DIR:-$HOME/.config/pai}"
if [ -f "$PAI_CHECK/Skills/CORE/SKILL.md" ]; then
  echo "kai-core-install: INSTALLED"
else
  echo "ERROR: kai-core-install not found (required)"
fi

if [ -f "$PAI_CHECK/Skills/Agents/SKILL.md" ]; then
  echo "kai-agents-skill: INSTALLED (escalation available)"
else
  echo "kai-agents-skill: NOT INSTALLED (escalation disabled, pack still works)"
fi

# 4. Check for existing Research skill
if [ -d "$PAI_CHECK/Skills/Research" ]; then
  echo "WARNING: Research skill already exists - will be replaced"
fi
```

---

### Step 1: Create Directory Structure

```bash
PAI_DIR="${PAI_DIR:-$HOME/.config/pai}"

mkdir -p "$PAI_DIR/Skills/Research/Data"
mkdir -p "$PAI_DIR/Skills/Research/Tools"
mkdir -p "$PAI_DIR/Skills/Research/Workflows"

ls -la "$PAI_DIR/Skills/Research/"
```

---

### Step 2: Create SKILL.md

```bash
cat > "$PAI_DIR/Skills/Research/SKILL.md" << 'SKILL_EOF'
---
name: Research
description: Hybrid research routing that saves tokens. USE WHEN user asks research questions, lookups, current info, explanations, or comparisons. Routes to Gemini (free explanations) or Perplexity (current info), escalates complex queries to research agents.
---

# Research - Hybrid Research Routing

**Auto-routes when user asks:** what is, explain, latest, compare, research, how does...work

## Overview

The Research skill saves Claude tokens by routing queries to external tools:
- **Gemini CLI** - Free explanations with 2M context
- **Perplexity API** - Real-time web search with citations
- **Research agents** - Complex queries needing synthesis (via Agents skill)

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **QuickLookup** | Simple queries (what is, explain, latest) | `Workflows/QuickLookup.md` |
| **DeepResearch** | Complex queries (research + recommend) | `Workflows/DeepResearch.md` |

## Route Triggers

| User Says | What Happens |
|-----------|--------------|
| "What is GraphQL?" | Direct → Gemini |
| "Explain async/await" | Direct → Gemini |
| "Latest React 19 features" | Direct → Perplexity |
| "Compare Next.js vs Remix" | Direct → Perplexity |
| "Research auth options for my API and recommend" | Agent → Both tools |

## How Routing Works

**Simple queries** (direct call, ~300 tokens):
- "what is", "define", "explain" → Gemini
- "latest", "news", "2025" → Perplexity
- No personal context ("for my project")

**Complex queries** (spawn agent, ~1500 tokens):
- Contains "and recommend", "and decide"
- Has personal context ("for my", "in my case")
- Multiple requirements in one query

## Token Savings

| Query Type | Without Skill | With Skill | Savings |
|------------|---------------|------------|---------|
| Simple lookup | ~1100 | ~400 | 63% |
| Current info | ~1100 | ~450 | 59% |
| Complex research | ~3000 | ~1500 | 50% |
SKILL_EOF
```

---

### Step 3: Create RoutingRules.yaml

```bash
cat > "$PAI_DIR/Skills/Research/Data/RoutingRules.yaml" << 'RULES_EOF'
# RoutingRules.yaml - Simple vs Complex query detection
# Determines whether to call tools directly or spawn a research agent

version: "1.0.0"

# ============================================================================
# SIMPLE QUERIES - Direct tool call (no agent overhead)
# ============================================================================
simple:
  # Patterns that indicate simple, direct queries
  patterns:
    - pattern: "^what (is|are|does)"
      description: "Definitional questions"
    - pattern: "^how (do|does|can|to)"
      description: "How-to questions"
    - pattern: "^explain"
      description: "Explanation requests"
    - pattern: "^define"
      description: "Definition requests"
    - pattern: "^describe"
      description: "Description requests"
    - pattern: "^(latest|recent|current|new)"
      description: "Current info requests"
    - pattern: "what.*(mean|meaning)"
      description: "Meaning questions"

  # Constraints for simple classification
  constraints:
    max_words: 20
    max_question_marks: 1
    no_personal_context: true  # "for my", "in my" makes it complex

# ============================================================================
# COMPLEX QUERIES - Spawn research agent
# ============================================================================
complex:
  # Patterns that require agent reasoning
  patterns:
    - pattern: "research.*(and|then)"
      description: "Research with follow-up action"
    - pattern: "(and|then)\\s+(recommend|decide|choose|suggest)"
      description: "Synthesis requests"
    - pattern: "compare.*(for my|for our|in my)"
      description: "Personalized comparison"
    - pattern: "analyze.*(implications|impact|effects)"
      description: "Impact analysis"
    - pattern: "investigate.*thoroughly"
      description: "Deep investigation"
    - pattern: "evaluate.*(options|alternatives)"
      description: "Option evaluation"

  # Signals that elevate to complex
  signals:
    personal_context:
      - "for my"
      - "in my"
      - "for our"
      - "my project"
      - "my app"
      - "my use case"
    synthesis_words:
      - "and recommend"
      - "and decide"
      - "and choose"
      - "and suggest"
      - "then implement"
    multi_part_indicators:
      - "first...then"
      - "investigate...analyze"
      - "research...recommend"

  # Thresholds
  thresholds:
    min_words_for_complex: 25
    multiple_questions: true  # More than one "?"

# ============================================================================
# FALLBACK
# ============================================================================
fallback:
  # If no patterns match, default to:
  default: "simple"
  default_tool: "gemini"
RULES_EOF
```

---

### Step 4: Create ToolRouting.yaml

```bash
cat > "$PAI_DIR/Skills/Research/Data/ToolRouting.yaml" << 'TOOL_EOF'
# ToolRouting.yaml - Gemini vs Perplexity selection
# Determines which external tool handles simple queries

version: "1.0.0"

# ============================================================================
# GEMINI - Best for explanations, technical depth
# ============================================================================
gemini:
  description: "Free explanations with 2M context window"
  use_when:
    patterns:
      - pattern: "^what (is|are)"
        weight: 20
        reason: "Conceptual definitions"
      - pattern: "^explain|^describe"
        weight: 25
        reason: "Deep explanations"
      - pattern: "how does.*work"
        weight: 20
        reason: "Technical understanding"
      - pattern: "\\b(syntax|code|function|class|method)\\b"
        weight: 15
        reason: "Code-related"
      - pattern: "\\b(architecture|design|pattern)\\b"
        weight: 15
        reason: "Technical architecture"
      - pattern: "\\b(concept|theory|principle)\\b"
        weight: 10
        reason: "Theoretical concepts"

    signals:
      - no_temporal_words: true  # No "latest", "new", "2025"
      - technical_depth: true    # Needs explanation, not just facts

  cli_command: "gemini"
  output_format: "json"

# ============================================================================
# PERPLEXITY - Best for current info, web research
# ============================================================================
perplexity:
  description: "Real-time web search with citations"
  use_when:
    patterns:
      - pattern: "\\b(latest|recent|current|new|2024|2025)\\b"
        weight: 30
        reason: "Needs current data"
      - pattern: "\\b(news|announcement|release|update)\\b"
        weight: 25
        reason: "Current events"
      - pattern: "\\b(compare|vs|versus|better|alternative)\\b"
        weight: 20
        reason: "Needs current opinions/benchmarks"
      - pattern: "\\b(price|pricing|cost|free|paid)\\b"
        weight: 15
        reason: "Current pricing info"
      - pattern: "\\b(best|popular|recommended|trending)\\b"
        weight: 15
        reason: "Current popularity/recommendations"
      - pattern: "\\b(statistics|stats|numbers|data)\\b"
        weight: 10
        reason: "Current statistics"

    signals:
      - needs_web_sources: true
      - temporal_sensitivity: true

  api_endpoint: "https://api.perplexity.ai/chat/completions"
  model: "sonar-pro"

# ============================================================================
# DECISION LOGIC
# ============================================================================
decision:
  # Score each tool based on pattern matches
  # Higher score wins
  minimum_score: 10
  tie_breaker: "gemini"  # If scores equal, prefer free option

  # Override rules
  overrides:
    # Always use Perplexity for these regardless of score
    force_perplexity:
      - "\\b(2024|2025)\\b"
      - "\\blatest\\b"
    # Always use Gemini for these
    force_gemini:
      - "\\bexplain.*(concept|theory)\\b"
      - "\\barchitecture.*design\\b"
TOOL_EOF
```

---

### Step 5: Create QueryRouter.ts

```bash
cat > "$PAI_DIR/Skills/Research/Tools/QueryRouter.ts" << 'ROUTER_EOF'
#!/usr/bin/env bun
/**
 * QueryRouter.ts - Research Query Analyzer and Router
 *
 * Analyzes queries and routes to appropriate handler:
 * - Simple queries → Direct tool call (GeminiQuery or PerplexityQuery)
 * - Complex queries → Spawn research agent
 *
 * Usage:
 *   bun run QueryRouter.ts --query "What is GraphQL?"
 *   bun run QueryRouter.ts --query "Research auth for my API and recommend"
 *   bun run QueryRouter.ts --query "Your query" --analyze  # Dry run
 */

import { parseArgs } from "util";
import { readFileSync, existsSync } from "fs";
import { parse as parseYaml } from "yaml";
import { spawn } from "bun";

const PAI_DIR = process.env.PAI_DIR || `${process.env.HOME}/.config/pai`;
const ROUTING_RULES = `${PAI_DIR}/Skills/Research/Data/RoutingRules.yaml`;
const TOOL_ROUTING = `${PAI_DIR}/Skills/Research/Data/ToolRouting.yaml`;

type Complexity = "simple" | "complex";
type Tool = "gemini" | "perplexity";

interface RoutingDecision {
  complexity: Complexity;
  tool: Tool | "agent";
  score: number;
  matchedPatterns: string[];
  reason: string;
}

function loadYaml(path: string): any {
  if (!existsSync(path)) {
    console.error(`Error: Config not found at ${path}`);
    process.exit(1);
  }
  return parseYaml(readFileSync(path, "utf-8"));
}

function analyzeComplexity(query: string, rules: any): { isComplex: boolean; reasons: string[] } {
  const queryLower = query.toLowerCase();
  const reasons: string[] = [];

  // Check complex patterns
  for (const rule of rules.complex?.patterns || []) {
    const regex = new RegExp(rule.pattern, "i");
    if (regex.test(query)) {
      reasons.push(`Pattern: ${rule.description}`);
    }
  }

  // Check complex signals
  const signals = rules.complex?.signals || {};

  for (const phrase of signals.personal_context || []) {
    if (queryLower.includes(phrase)) {
      reasons.push(`Personal context: "${phrase}"`);
    }
  }

  for (const phrase of signals.synthesis_words || []) {
    if (queryLower.includes(phrase)) {
      reasons.push(`Synthesis request: "${phrase}"`);
    }
  }

  // Check word count threshold
  const wordCount = query.split(/\s+/).length;
  const minWords = rules.complex?.thresholds?.min_words_for_complex || 25;
  if (wordCount > minWords) {
    reasons.push(`Long query: ${wordCount} words`);
  }

  // Check multiple questions
  const questionMarks = (query.match(/\?/g) || []).length;
  if (questionMarks > 1) {
    reasons.push(`Multiple questions: ${questionMarks}`);
  }

  return {
    isComplex: reasons.length > 0,
    reasons
  };
}

function selectTool(query: string, toolConfig: any): { tool: Tool; score: number; reasons: string[] } {
  const scores: Record<Tool, { score: number; reasons: string[] }> = {
    gemini: { score: 0, reasons: [] },
    perplexity: { score: 0, reasons: [] }
  };

  // Check overrides first
  const overrides = toolConfig.decision?.overrides || {};

  for (const pattern of overrides.force_perplexity || []) {
    if (new RegExp(pattern, "i").test(query)) {
      return { tool: "perplexity", score: 100, reasons: ["Override: force_perplexity"] };
    }
  }

  for (const pattern of overrides.force_gemini || []) {
    if (new RegExp(pattern, "i").test(query)) {
      return { tool: "gemini", score: 100, reasons: ["Override: force_gemini"] };
    }
  }

  // Score Gemini patterns
  for (const rule of toolConfig.gemini?.use_when?.patterns || []) {
    const regex = new RegExp(rule.pattern, "i");
    if (regex.test(query)) {
      scores.gemini.score += rule.weight;
      scores.gemini.reasons.push(rule.reason);
    }
  }

  // Score Perplexity patterns
  for (const rule of toolConfig.perplexity?.use_when?.patterns || []) {
    const regex = new RegExp(rule.pattern, "i");
    if (regex.test(query)) {
      scores.perplexity.score += rule.weight;
      scores.perplexity.reasons.push(rule.reason);
    }
  }

  // Determine winner
  const minScore = toolConfig.decision?.minimum_score || 10;
  const tieBreaker = toolConfig.decision?.tie_breaker || "gemini";

  if (scores.gemini.score === scores.perplexity.score) {
    return {
      tool: tieBreaker as Tool,
      score: scores[tieBreaker as Tool].score,
      reasons: scores[tieBreaker as Tool].reasons.length > 0
        ? scores[tieBreaker as Tool].reasons
        : ["Tie-breaker default"]
    };
  }

  const winner: Tool = scores.gemini.score > scores.perplexity.score ? "gemini" : "perplexity";
  return {
    tool: winner,
    score: scores[winner].score,
    reasons: scores[winner].reasons
  };
}

function routeQuery(query: string): RoutingDecision {
  const routingRules = loadYaml(ROUTING_RULES);
  const toolConfig = loadYaml(TOOL_ROUTING);

  // Step 1: Check complexity
  const { isComplex, reasons: complexReasons } = analyzeComplexity(query, routingRules);

  if (isComplex) {
    return {
      complexity: "complex",
      tool: "agent",
      score: 0,
      matchedPatterns: complexReasons,
      reason: "Complex query - spawn research agent"
    };
  }

  // Step 2: Select tool for simple query
  const { tool, score, reasons: toolReasons } = selectTool(query, toolConfig);

  return {
    complexity: "simple",
    tool,
    score,
    matchedPatterns: toolReasons,
    reason: `Simple query - direct ${tool} call`
  };
}

async function executeQuery(query: string, decision: RoutingDecision): Promise<string> {
  if (decision.tool === "agent") {
    // Check if Agents skill is available
    const agentsSkill = `${PAI_DIR}/Skills/Agents/Tools/AgentFactory.ts`;
    if (existsSync(agentsSkill)) {
      return `[Research Skill] Complex query detected. Spawn research agent with:\n` +
             `bun run ${agentsSkill} --traits "research,analytical,thorough" --task "${query}"`;
    } else {
      // Fallback: use Perplexity for complex queries without Agents skill
      const proc = spawn(["bun", "run", `${PAI_DIR}/Skills/Research/Tools/PerplexityQuery.ts`, "--query", query], {
        stdout: "pipe",
        stderr: "pipe"
      });
      return await new Response(proc.stdout).text();
    }
  }

  // Execute direct tool call
  const toolScript = decision.tool === "gemini"
    ? `${PAI_DIR}/Skills/Research/Tools/GeminiQuery.ts`
    : `${PAI_DIR}/Skills/Research/Tools/PerplexityQuery.ts`;

  const proc = spawn(["bun", "run", toolScript, "--query", query], {
    stdout: "pipe",
    stderr: "pipe"
  });

  return await new Response(proc.stdout).text();
}

async function main() {
  const { values } = parseArgs({
    args: Bun.argv.slice(2),
    options: {
      query: { type: "string", short: "q" },
      analyze: { type: "boolean", short: "a" },
      output: { type: "string", short: "o", default: "text" },
      help: { type: "boolean", short: "h" }
    }
  });

  if (values.help || !values.query) {
    console.log(`
QueryRouter - Research Query Analyzer and Router

USAGE:
  bun run QueryRouter.ts --query "Your question"

OPTIONS:
  -q, --query     The query to route
  -a, --analyze   Analyze only, don't execute
  -o, --output    Output format: text (default), json
  -h, --help      Show this help

EXAMPLES:
  bun run QueryRouter.ts -q "What is GraphQL?"
  bun run QueryRouter.ts -q "Latest React 19 features"
  bun run QueryRouter.ts -q "Research auth for my API and recommend" --analyze
`);
    return;
  }

  const decision = routeQuery(values.query);

  if (values.analyze) {
    if (values.output === "json") {
      console.log(JSON.stringify(decision, null, 2));
    } else {
      console.log("=== Query Analysis ===\n");
      console.log(`Query: ${values.query}`);
      console.log(`Complexity: ${decision.complexity.toUpperCase()}`);
      console.log(`Route to: ${decision.tool.toUpperCase()}`);
      console.log(`Score: ${decision.score}`);
      console.log(`Patterns: ${decision.matchedPatterns.join(", ") || "none"}`);
      console.log(`Reason: ${decision.reason}`);
    }
    return;
  }

  // Execute
  const result = await executeQuery(values.query, decision);
  console.log(result);
}

main().catch(console.error);
ROUTER_EOF
```

---

### Step 6: Create GeminiQuery.ts

```bash
cat > "$PAI_DIR/Skills/Research/Tools/GeminiQuery.ts" << 'GEMINI_EOF'
#!/usr/bin/env bun
/**
 * GeminiQuery.ts - Gemini CLI Wrapper
 *
 * Calls Gemini CLI and returns structured output.
 *
 * Usage:
 *   bun run GeminiQuery.ts --query "What is GraphQL?"
 */

import { parseArgs } from "util";
import { spawn } from "bun";

interface GeminiResponse {
  success: boolean;
  query: string;
  response: string;
  source: "gemini";
  error?: string;
}

async function queryGemini(query: string): Promise<GeminiResponse> {
  try {
    const proc = spawn(["gemini", "-o", "json", query], {
      stdout: "pipe",
      stderr: "pipe"
    });

    const output = await new Response(proc.stdout).text();
    const exitCode = await proc.exited;

    if (exitCode !== 0) {
      const stderr = await new Response(proc.stderr).text();
      return {
        success: false,
        query,
        response: "",
        source: "gemini",
        error: stderr || "Gemini CLI failed"
      };
    }

    // Parse Gemini JSON output
    try {
      const json = JSON.parse(output);
      const responseText = json.text || json.response || json.content || output;
      return {
        success: true,
        query,
        response: responseText,
        source: "gemini"
      };
    } catch {
      // If not JSON, return raw output
      return {
        success: true,
        query,
        response: output.trim(),
        source: "gemini"
      };
    }
  } catch (error) {
    return {
      success: false,
      query,
      response: "",
      source: "gemini",
      error: `Gemini error: ${(error as Error).message}`
    };
  }
}

async function main() {
  const { values } = parseArgs({
    args: Bun.argv.slice(2),
    options: {
      query: { type: "string", short: "q" },
      output: { type: "string", short: "o", default: "text" },
      help: { type: "boolean", short: "h" }
    }
  });

  if (values.help || !values.query) {
    console.log(`
GeminiQuery - Query Gemini CLI

USAGE:
  bun run GeminiQuery.ts --query "Your question"

OPTIONS:
  -q, --query   The query to send to Gemini
  -o, --output  Output format: text (default), json
  -h, --help    Show this help
`);
    return;
  }

  const result = await queryGemini(values.query);

  if (values.output === "json") {
    console.log(JSON.stringify(result, null, 2));
  } else {
    if (result.success) {
      console.log(result.response);
    } else {
      console.error(`Error: ${result.error}`);
      process.exit(1);
    }
  }
}

main().catch(console.error);
GEMINI_EOF
```

---

### Step 7: Create PerplexityQuery.ts

```bash
cat > "$PAI_DIR/Skills/Research/Tools/PerplexityQuery.ts" << 'PERPLEXITY_EOF'
#!/usr/bin/env bun
/**
 * PerplexityQuery.ts - Perplexity API Client
 *
 * Calls Perplexity API for real-time web research.
 *
 * Usage:
 *   bun run PerplexityQuery.ts --query "Latest React 19 features"
 */

import { parseArgs } from "util";
import { readFileSync } from "fs";

const PAI_DIR = process.env.PAI_DIR || `${process.env.HOME}/.config/pai`;

interface PerplexityResponse {
  success: boolean;
  query: string;
  response: string;
  source: "perplexity";
  citations?: string[];
  error?: string;
}

function getApiKey(): string {
  // Try environment variable first
  if (process.env.PERPLEXITY_API_KEY) {
    return process.env.PERPLEXITY_API_KEY;
  }

  // Try .env file
  try {
    const envPath = `${PAI_DIR}/.env`;
    const envContent = readFileSync(envPath, "utf-8");
    const match = envContent.match(/PERPLEXITY_API_KEY=["']?([^"'\n]+)["']?/);
    if (match) {
      return match[1];
    }
  } catch {
    // Ignore
  }

  return "";
}

async function queryPerplexity(query: string): Promise<PerplexityResponse> {
  const apiKey = getApiKey();

  if (!apiKey) {
    return {
      success: false,
      query,
      response: "",
      source: "perplexity",
      error: "PERPLEXITY_API_KEY not set. Add to $PAI_DIR/.env or environment."
    };
  }

  try {
    const response = await fetch("https://api.perplexity.ai/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${apiKey}`
      },
      body: JSON.stringify({
        model: "sonar-pro",
        messages: [
          {
            role: "system",
            content: "You are a research assistant. Provide accurate, well-sourced information. Include relevant citations when available. Be concise but comprehensive."
          },
          {
            role: "user",
            content: query
          }
        ]
      })
    });

    if (!response.ok) {
      return {
        success: false,
        query,
        response: "",
        source: "perplexity",
        error: `Perplexity API error: ${response.status} ${response.statusText}`
      };
    }

    const data = await response.json();
    const content = data.choices?.[0]?.message?.content || "";
    const citations = data.citations || [];

    return {
      success: true,
      query,
      response: content,
      source: "perplexity",
      citations
    };
  } catch (error) {
    return {
      success: false,
      query,
      response: "",
      source: "perplexity",
      error: `Perplexity error: ${(error as Error).message}`
    };
  }
}

async function main() {
  const { values } = parseArgs({
    args: Bun.argv.slice(2),
    options: {
      query: { type: "string", short: "q" },
      output: { type: "string", short: "o", default: "text" },
      help: { type: "boolean", short: "h" }
    }
  });

  if (values.help || !values.query) {
    console.log(`
PerplexityQuery - Query Perplexity API

USAGE:
  bun run PerplexityQuery.ts --query "Your question"

OPTIONS:
  -q, --query   The query to send to Perplexity
  -o, --output  Output format: text (default), json
  -h, --help    Show this help

ENVIRONMENT:
  PERPLEXITY_API_KEY - Your Perplexity API key (or set in $PAI_DIR/.env)
`);
    return;
  }

  const result = await queryPerplexity(values.query);

  if (values.output === "json") {
    console.log(JSON.stringify(result, null, 2));
  } else {
    if (result.success) {
      console.log(result.response);
      if (result.citations && result.citations.length > 0) {
        console.log("\n---\nSources:");
        result.citations.forEach((c, i) => console.log(`${i + 1}. ${c}`));
      }
    } else {
      console.error(`Error: ${result.error}`);
      process.exit(1);
    }
  }
}

main().catch(console.error);
PERPLEXITY_EOF
```

---

### Step 8: Create Workflows

#### QuickLookup.md

```bash
cat > "$PAI_DIR/Skills/Research/Workflows/QuickLookup.md" << 'QUICK_EOF'
# QuickLookup Workflow

**Handles simple research queries with direct tool calls.**

## When to Use

Query is classified as "simple" by QueryRouter:
- Starts with "what is", "explain", "define"
- Contains "latest", "news", "current"
- No personal context ("for my project")
- Short query (< 20 words)

## Steps

### Step 1: Run QueryRouter

```bash
bun run $PAI_DIR/Skills/Research/Tools/QueryRouter.ts --query "USER_QUERY"
```

QueryRouter:
1. Analyzes complexity → Simple
2. Selects tool (Gemini or Perplexity)
3. Executes tool and returns result

### Step 2: Return Result

Result is returned directly to Claude context.

## Tool Selection

| Query Type | Tool | Why |
|------------|------|-----|
| "What is X?" | Gemini | Conceptual explanation |
| "Explain Y" | Gemini | Technical depth |
| "Latest Z" | Perplexity | Current info needed |
| "Compare A vs B" | Perplexity | Needs current opinions |

## Token Cost

~300-450 tokens (direct tool call, minimal Claude involvement)
QUICK_EOF
```

#### DeepResearch.md

```bash
cat > "$PAI_DIR/Skills/Research/Workflows/DeepResearch.md" << 'DEEP_EOF'
# DeepResearch Workflow

**Handles complex research queries by spawning research agents.**

## When to Use

Query is classified as "complex" by QueryRouter:
- Contains "and recommend", "and decide"
- Has personal context ("for my project")
- Multiple questions in one query
- Long query with multiple requirements

## Steps

### Step 1: Run QueryRouter

```bash
bun run $PAI_DIR/Skills/Research/Tools/QueryRouter.ts --query "USER_QUERY" --analyze
```

QueryRouter identifies complexity signals:
- Personal context detected
- Synthesis request detected
- Multiple requirements

### Step 2: Check Agents Skill

If `kai-agents-skill` is installed:

```bash
bun run $PAI_DIR/Skills/Agents/Tools/AgentFactory.ts \
  --traits "research,analytical,thorough" \
  --task "USER_QUERY"
```

Agent spawns with:
- Expertise: research
- Personality: analytical
- Approach: thorough

### Step 3: Agent Uses Research Tools

The research agent can call:
- `GeminiQuery.ts` - For explanations
- `PerplexityQuery.ts` - For current info

Agent combines results intelligently.

### Step 4: If No Agents Skill

Fallback: Query goes to Perplexity directly.
Less intelligent but still provides current info.

## Token Cost

~1500 tokens (agent spawn + tool calls + synthesis)

## Example

```
User: "Research authentication approaches for my Node.js API and recommend one"

QueryRouter: Complex (personal context + "and recommend")
→ Spawn research agent
→ Agent calls Perplexity: "current auth best practices Node.js 2025"
→ Agent calls Gemini: "explain JWT vs session auth architecture"
→ Agent synthesizes: "For your Node.js API, I recommend..."
```
DEEP_EOF
```

---

### Step 9: Install Dependencies

```bash
cd "$PAI_DIR/Skills/Research/Tools"
bun add yaml
```

---

### Step 10: Add API Key to Environment

```bash
# Add to your PAI .env file
echo 'PERPLEXITY_API_KEY="your_key_here"' >> "$PAI_DIR/.env"
```

Replace `your_key_here` with your actual Perplexity API key.

---

### Step 11: Verify Installation

```bash
PAI_DIR="${PAI_DIR:-$HOME/.config/pai}"

# Check all files exist
echo "Checking installation..."
[ -f "$PAI_DIR/Skills/Research/SKILL.md" ] && echo "SKILL.md" || echo "MISSING: SKILL.md"
[ -f "$PAI_DIR/Skills/Research/Data/RoutingRules.yaml" ] && echo "RoutingRules.yaml" || echo "MISSING"
[ -f "$PAI_DIR/Skills/Research/Data/ToolRouting.yaml" ] && echo "ToolRouting.yaml" || echo "MISSING"
[ -f "$PAI_DIR/Skills/Research/Tools/QueryRouter.ts" ] && echo "QueryRouter.ts" || echo "MISSING"
[ -f "$PAI_DIR/Skills/Research/Tools/GeminiQuery.ts" ] && echo "GeminiQuery.ts" || echo "MISSING"
[ -f "$PAI_DIR/Skills/Research/Tools/PerplexityQuery.ts" ] && echo "PerplexityQuery.ts" || echo "MISSING"

# Test QueryRouter analysis
echo ""
echo "Testing QueryRouter..."
bun run "$PAI_DIR/Skills/Research/Tools/QueryRouter.ts" \
  --query "What is GraphQL?" \
  --analyze

# Test with complex query
echo ""
echo "Testing complex query detection..."
bun run "$PAI_DIR/Skills/Research/Tools/QueryRouter.ts" \
  --query "Research authentication options for my API and recommend" \
  --analyze
```

Expected output: All files exist, QueryRouter correctly identifies simple vs complex.

---

## Invocation Scenarios

The Research skill triggers automatically when Claude detects research-related patterns:

| User Says | Classification | Tool | Tokens |
|-----------|---------------|------|--------|
| "What is GraphQL?" | Simple | Gemini | ~400 |
| "Explain async/await" | Simple | Gemini | ~400 |
| "Latest React 19 features" | Simple | Perplexity | ~450 |
| "Compare Next.js vs Remix" | Simple | Perplexity | ~400 |
| "Research auth and recommend for my API" | Complex | Agent | ~1500 |

---

## Example Usage

### Example 1: Simple Explanation

```
User: "What is WebAssembly?"

Research skill activates:
1. QueryRouter: Simple, Gemini
2. GeminiQuery runs
3. Result: Technical explanation of WebAssembly

Tokens: ~400 (vs ~1100 without pack)
```

### Example 2: Current Information

```
User: "What are the latest features in Bun 2.0?"

Research skill activates:
1. QueryRouter: Simple, Perplexity (contains "latest")
2. PerplexityQuery runs
3. Result: Current Bun 2.0 features with citations

Tokens: ~450 (vs ~1100 without pack)
```

### Example 3: Complex Research

```
User: "Research API authentication approaches for my enterprise Node.js
      application and recommend the best option"

Research skill activates:
1. QueryRouter: Complex (personal context + "and recommend")
2. AgentFactory spawns research agent
3. Agent calls Perplexity: Current auth best practices
4. Agent calls Gemini: Technical architecture comparison
5. Agent synthesizes recommendation

Tokens: ~1500 (vs ~3000 without pack)
```

---

## Configuration

### Environment Variables

**Required:**
```bash
# Add to $PAI_DIR/.env
PERPLEXITY_API_KEY="pplx-xxxxx"
```

**Optional:**
```bash
# Gemini CLI uses its own auth (already configured if gemini works)
```

### Customizing Routing Rules

Edit `$PAI_DIR/Skills/Research/Data/RoutingRules.yaml`:
- Add patterns to `simple.patterns` for direct routing
- Add signals to `complex.signals` for agent escalation

Edit `$PAI_DIR/Skills/Research/Data/ToolRouting.yaml`:
- Adjust pattern weights to favor Gemini or Perplexity
- Add `force_gemini` or `force_perplexity` overrides

---

## Customization

### Recommended Customization

**What to Customize:** Tool preference weights in ToolRouting.yaml

**Why:** Your research patterns may favor current info (Perplexity) or deep explanations (Gemini). Adjust weights to match your usage.

**Process:**
1. Use the skill for a week, noting which tool you wish it had picked
2. Edit `ToolRouting.yaml` to increase weights for preferred patterns
3. Add force overrides for consistent cases

**Expected Outcome:** Routing matches your research style, maximizing token savings.

---

### Optional Customization

| Customization | File | Impact |
|--------------|------|--------|
| Add complexity signals | RoutingRules.yaml | More queries escalate to agents |
| Change Perplexity model | PerplexityQuery.ts | Different speed/quality tradeoff |
| Add new patterns | ToolRouting.yaml | Better tool selection |
| Adjust thresholds | RoutingRules.yaml | Tune simple vs complex boundary |

---

## Credits

- **Original concept**: Dusty - PAI research optimization
- **Pattern system**: Inspired by kai-agents-skill trait routing
- **Gemini integration**: Uses official Gemini CLI
- **Perplexity integration**: Uses Perplexity API

---

## Related Work

- [kai-agents-skill](kai-agents-skill.md) - Provides agent escalation for complex queries
- [kai-core-install](kai-core-install.md) - Skill routing infrastructure

---

## Works Well With

- **kai-agents-skill** - Complex queries spawn research agents with full trait system
- **kai-prompting-skill** - Could format research prompts with templates

---

## Relationships

### Child Of
- kai-core-install (skill routing infrastructure)

### Sibling Of
- kai-agents-skill (shares agent patterns)

### Part Of Collection
- Kai Bundle

---

## Changelog

### 1.0.0 - 2025-12-30
- Initial release
- Hybrid routing: simple → direct, complex → agent
- Gemini CLI integration for explanations
- Perplexity API integration for current info
- YAML-configurable routing rules
- 50-63% token savings vs baseline
