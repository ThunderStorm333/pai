---
name: Kai Skill Suggester
pack-id: danielmiessler-kai-skill-suggester-core-v1.0.0
version: 1.0.0
author: danielmiessler
description: History-driven skill discovery system that mines usage patterns to suggest new skills and updates to existing skills
type: skill
purpose-type: [productivity, automation, development]
platform: claude-code
dependencies:
  - danielmiessler-kai-core-install-core-v1.0.0
  - danielmiessler-kai-history-system-core-v1.0.0
keywords: [skills, suggestions, patterns, analysis, history, mining, automation, discovery, optimization, gaps, recommendations, learning]
---

<p align="center">
  <img src="icons/kai-skill-suggester.png" alt="Kai Skill Suggester" width="256">
</p>

# Kai Skill Suggester

> History-driven skill discovery system that mines your usage patterns to suggest new skills and updates to existing skills

## Installation Prompt

You are receiving a PAI Pack - a modular upgrade for AI agent systems.

**What is PAI?** See: [PAI Project Overview](../README.md#what-is-pai)

**What is a Pack?** See: [Pack System](../README.md#the-journey-pai-v1x--v20)

This pack adds intelligent skill discovery to your PAI infrastructure. The Skill Suggester analyzes your history to find patterns worth turning into skills:

- **Pattern Mining**: Detects repeated request patterns across sessions
- **Gap Detection**: Identifies requests that don't match existing skills
- **Update Suggestions**: Proposes improvements to existing skills based on actual usage
- **Zero Effort**: Works entirely from your existing history - no manual tagging required

**Core principle:** Your usage patterns reveal what skills you need.

The system presents a summary first ("3 new skill opportunities, 2 updates"), then lets you drill down into details and approve proposals. When approved, it delegates to the existing CreateSkill workflow.

Please follow the installation instructions below to integrate this pack into your infrastructure.

---

## What's Included

| Component | File | Purpose |
|-----------|------|---------|
| Main Skill | `Skills/SkillSuggester/SKILL.md` | Skill definition with conversational triggers |
| CLI Entry Point | `Skills/SkillSuggester/Tools/SkillSuggest.ts` | Main command-line interface |
| History Miner | `Skills/SkillSuggester/Tools/HistoryMiner.ts` | JSONL parser with chunked processing |
| Pattern Clustering | `Skills/SkillSuggester/Tools/PatternClustering.ts` | Groups similar requests |
| Gap Detector | `Skills/SkillSuggester/Tools/GapDetector.ts` | Compares patterns vs skill coverage |
| Update Analyzer | `Skills/SkillSuggester/Tools/UpdateAnalyzer.ts` | Detects skills needing updates |
| Proposal Formatter | `Skills/SkillSuggester/Tools/ProposalFormatter.ts` | Generates summary and side-by-side output |
| Analysis Config | `Skills/SkillSuggester/AnalysisConfig.md` | Configurable thresholds |
| Pattern Heuristics | `Skills/SkillSuggester/Data/PatternHeuristics.yaml` | "What counts as a skill" rules |
| Update Signals | `Skills/SkillSuggester/Data/UpdateSignals.yaml` | "What counts as an update" rules |
| Analyze Workflow | `Skills/SkillSuggester/Workflows/AnalyzeHistory.md` | Main analysis workflow |
| Apply Workflow | `Skills/SkillSuggester/Workflows/ApplyProposal.md` | Execute approved proposals |

**Summary:**
- **Files created:** 12
- **Skills registered:** 1 (SkillSuggester)
- **Dependencies:** kai-core-install (CreateSkill), kai-history-system (raw-outputs)

---

## The Concept and/or Problem

Skills in PAI are powerful abstractions - they bundle workflows, tools, and context for specific domains. But how do you know what skills you need?

**The Discovery Problem:**

Most users create skills reactively:
- "I keep doing the same Docker setup, maybe I should make a skill"
- "I've explained this workflow three times, I should capture it"
- Weeks later: "Wait, I already solved this problem last month"

This reactive approach has problems:
- **Missed opportunities**: Patterns go unnoticed until frustration hits
- **Inconsistent coverage**: Some domains get skills, others don't
- **Stale skills**: Usage patterns evolve but skills don't
- **Manual effort**: You have to remember AND act on patterns

**The Update Problem:**

Existing skills also decay:
- Triggers don't match how you actually phrase requests
- Workflows have evolved but the skill hasn't
- New use cases emerged that the skill should cover
- Some skills are never used (should be deprecated)

**The Core Issue:**

You already have the data to answer "what skills do I need?" - it's in your history. Every request, every tool sequence, every session is captured. But manually analyzing hundreds of JSONL entries is impractical.

---

## The Solution

The Skill Suggester mines your history automatically to discover skill opportunities and propose updates.

**Analysis Pipeline:**

```
History Data → Pattern Extraction → Clustering → Gap Detection → Proposal Generation
```

1. **History Mining**: Parses `raw-outputs/*.jsonl` from the history system
2. **Pattern Clustering**: Groups similar requests by keywords and tool sequences
3. **Gap Detection**: Compares patterns against `skill-index.json` triggers
4. **Update Detection**: Identifies skills with mismatched triggers or evolved workflows
5. **Proposal Generation**: Creates structured proposals with confidence scores

**Heuristics for "What Counts as a Skill":**

| Signal | Why It Matters |
|--------|---------------|
| Repeated request (3+ times, 2+ sessions) | Indicates ongoing need |
| Multi-step workflow sequence | Complex enough to benefit from automation |
| External tool/API integration | Integration patterns are reusable |
| Context bundles (same files together) | Information that belongs together |
| Low-confidence skill match | Gap in current coverage |

**Integration with Existing System:**

When you approve a proposal, SkillSuggester delegates to:
- **CreateSkill workflow** → scaffolds the new skill
- **GenerateSkillIndex.ts** → refreshes the skill index
- **ValidateSkill workflow** → ensures structure compliance

No new skill creation mechanism - it uses what already exists.

---

## What Makes This Different

This sounds similar to auto-tagging systems which also detect patterns. What makes this approach different?

The Skill Suggester operates at the semantic skill layer, not the content layer. It doesn't just find repeated text - it identifies patterns that map to the 5-tier PAI skill architecture (frontmatter, body, context, workflows, tools) and generates structured proposals that integrate with existing creation workflows.

- Skills are proposed, never auto-created without consent
- Confidence scores explain why each suggestion was made
- Side-by-side diffs show exactly what would change
- Delegates to CreateSkill rather than reimplementing creation

---

## Installation

### Prerequisites

- **Bun runtime**: `curl -fsSL https://bun.sh/install | bash`
- **Claude Code** (or compatible agent system with hook support)
- **kai-core-install** pack installed (for CreateSkill, skill-index.json)
- **kai-history-system** pack installed (for raw-outputs data)

---

### Pre-Installation: System Analysis

#### Step 0.1: Detect Current Configuration

```bash
# 1. Check if PAI_DIR is set
echo "PAI_DIR: ${PAI_DIR:-'NOT SET - will use ~/.config/pai'}"

# 2. Check for existing PAI directory
PAI_CHECK="${PAI_DIR:-$HOME/.config/pai}"
if [ -d "$PAI_CHECK" ]; then
  echo "PAI directory EXISTS at: $PAI_CHECK"
else
  echo "PAI directory does not exist (clean install)"
fi

# 3. Check for existing SkillSuggester skill
if [ -d "$PAI_CHECK/Skills/SkillSuggester" ]; then
  echo "WARNING: SkillSuggester skill already exists"
else
  echo "SkillSuggester skill does not exist (will be created)"
fi

# 4. Check for history system
if [ -d "$PAI_CHECK/history/raw-outputs" ]; then
  echo "History system detected"
  ls -la "$PAI_CHECK/history/raw-outputs/" 2>/dev/null | head -5
else
  echo "WARNING: No history data found - system needs usage before suggestions work"
fi

# 5. Check for skill-index.json
if [ -f "$PAI_CHECK/Skills/skill-index.json" ]; then
  echo "skill-index.json exists"
else
  echo "WARNING: skill-index.json not found - run GenerateSkillIndex.ts first"
fi
```

#### Step 0.2: Verify Dependencies

```bash
PAI_CHECK="${PAI_DIR:-$HOME/.config/pai}"

# Check for kai-core-install (provides CreateSkill)
if [ -d "$PAI_CHECK/Skills/CreateSkill" ]; then
  echo "kai-core-install is installed (CreateSkill found)"
else
  echo "ERROR: kai-core-install not installed - required dependency"
  exit 1
fi

# Check for kai-history-system (provides raw-outputs)
if [ -d "$PAI_CHECK/history" ]; then
  echo "kai-history-system is installed (history directory found)"
else
  echo "ERROR: kai-history-system not installed - required dependency"
  exit 1
fi
```

#### Step 0.3: Conflict Resolution Matrix

| Scenario | Existing State | Action |
|----------|---------------|--------|
| **Clean Install** | No SkillSuggester directory | Proceed normally with Step 1 |
| **Skill Exists** | SkillSuggester already present | Backup old version, then replace |
| **No History** | raw-outputs empty | Install anyway, warn user to use system first |
| **Missing Dependencies** | CreateSkill or history missing | Install dependencies first |

---

### Step 1: Create Directory Structure

```bash
PAI_DIR="${PAI_DIR:-$HOME/.config/pai}"
mkdir -p "$PAI_DIR/Skills/SkillSuggester/Workflows"
mkdir -p "$PAI_DIR/Skills/SkillSuggester/Tools"
mkdir -p "$PAI_DIR/Skills/SkillSuggester/Data"

# Verify structure
ls -la "$PAI_DIR/Skills/SkillSuggester/"
```

---

### Step 2: Create SKILL.md

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/SKILL.md" << 'SKILL_EOF'
---
name: SkillSuggester
description: Analyze history patterns to suggest new skills and skill updates. USE WHEN suggest skills, skill gaps, analyze patterns, what skills should I create, skill recommendations, improve my skills.
---

# Skill Suggester

Mines PAI history to discover repeated patterns worthy of becoming skills, and identifies existing skills that need updates based on usage patterns.

## Quick Start

**Conversational:**
- "Suggest some skills for me"
- "What skills should I create?"
- "Analyze my history for patterns"
- "Any skill gaps?"

**CLI:**
```bash
bun run $PAI_DIR/Skills/SkillSuggester/Tools/SkillSuggest.ts --days 30
```

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **AnalyzeHistory** | "suggest skills", "analyze patterns" | `Workflows/AnalyzeHistory.md` |
| **ApplyProposal** | "create [SkillName]", "update [SkillName]" | `Workflows/ApplyProposal.md` |

## Output Format

**Summary first**, then drill-down on request:

```
Skill Analysis Complete (last 30 days)

**New Skill Opportunities:** N found
  1. SkillName (confidence%) - occurrences

**Existing Skill Updates:** N suggested
  1. SkillName - change type (confidence%)

Say "show me [name]" for details, or "create [name]" to proceed.
```

## Examples

**Example 1: Basic suggestion request**
```
User: "Suggest some skills for me"

SkillSuggester:
Skill Analysis Complete (last 30 days)

**New Skill Opportunities:** 2 found
  1. DockerWorkflow (87%) - 12 occurrences across 5 sessions
  2. DatabaseMigrations (72%) - 8 occurrences across 3 sessions

**Existing Skill Updates:** 1 suggested
  1. Art - add triggers: infographics, thumbnails (82%)

Say "show me DockerWorkflow" for details, or "create DockerWorkflow" to proceed.
```

**Example 2: Drill down into proposal**
```
User: "Show me DockerWorkflow"

SkillSuggester:
## DockerWorkflow
**Confidence:** 87% | **Evidence:** 12 occurrences across 5 sessions

**Proposed Triggers:** docker, container, dockerfile, compose, build image

**Suggested Workflows:**
- BuildImage (8 occurrences)
- DeployContainer (6 occurrences)

**Recent Evidence:**
- 2025-12-28: "build a docker image for the api"
- 2025-12-27: "deploy the container to staging"

Say "create DockerWorkflow" to generate this skill.
```

**Example 3: Approve and create**
```
User: "Create DockerWorkflow"

SkillSuggester:
Creating DockerWorkflow skill...
[Delegates to CreateSkill workflow]

Created: $PAI_DIR/Skills/DockerWorkflow/
- SKILL.md with triggers
- Workflows/BuildImage.md
- Workflows/DeployContainer.md
- Tools/ directory

Skill index updated. DockerWorkflow is now available.
```
SKILL_EOF
```

---

### Step 3: Create Analysis Config

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/AnalysisConfig.md" << 'CONFIG_EOF'
# Skill Suggester Configuration

## Thresholds

| Parameter | Default | Description |
|-----------|---------|-------------|
| `minOccurrences` | 3 | Minimum times a pattern must appear |
| `minSessions` | 2 | Minimum unique sessions |
| `minScore` | 5.0 | Minimum worthiness score to suggest |
| `maxEventsPerAnalysis` | 50000 | Cap on events for token efficiency |
| `similarityThreshold` | 0.7 | Minimum similarity to cluster patterns |
| `triggerMatchThreshold` | 30 | Below this = gap detected |
| `defaultDays` | 30 | Default analysis period |

## Adjusting for Your Usage

**High-volume users (10+ sessions/day):**
- Increase `minOccurrences` to 5
- Increase `minSessions` to 3

**Low-volume users (<1 session/day):**
- Decrease `minOccurrences` to 2
- Decrease `minScore` to 3.0
- Increase `defaultDays` to 60

## Event Types Analyzed

```yaml
analyzedEventTypes:
  - UserPromptSubmit    # User requests (primary source)
  - PostToolUse         # Tool completion (for workflow detection)
```

Excluded: PreToolUse (too noisy), SessionStart/End (not useful for patterns)
CONFIG_EOF
```

---

### Step 4: Create Data Files

#### 4.1: Pattern Heuristics

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Data/PatternHeuristics.yaml" << 'HEURISTICS_EOF'
# Pattern Heuristics: What counts as worth a skill
# Each heuristic contributes to a pattern's score

heuristics:
  repeated_request:
    description: Same request pattern appears across multiple sessions
    min_occurrences: 3
    min_sessions: 2
    weight: 1.0

  multi_step_workflow:
    description: Sequence of 3+ tool calls that repeats
    min_steps: 3
    min_occurrences: 2
    weight: 1.5
    tool_sequence_match_threshold: 0.7

  external_integration:
    description: External tool/API usage that repeats
    indicators:
      - curl
      - wget
      - fetch
      - api
      - endpoint
    min_occurrences: 2
    weight: 1.2

  context_bundle:
    description: Same files/info requested together repeatedly
    min_files: 2
    min_occurrences: 3
    weight: 0.8

  decision_pattern:
    description: Conditional logic that repeats
    indicators:
      - if
      - when
      - unless
      - depending
      - based on
    with_action_verbs: true
    min_occurrences: 3
    weight: 1.0

  low_confidence_match:
    description: Requests with low match scores against existing skills
    max_skill_match_score: 30
    min_occurrences: 2
    weight: 0.9

scoring:
  threshold: 5.0
  formula: "(occurrences * weight) + (session_count * 0.5) + complexity_bonus"
HEURISTICS_EOF
```

#### 4.2: Update Signals

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Data/UpdateSignals.yaml" << 'SIGNALS_EOF'
# Update Signals: What counts as worth an update to existing skill

signals:
  trigger_mismatch:
    description: Actual usage patterns don't match USE WHEN triggers
    detection:
      - Pattern matches skill by description but not by triggers
      - New keywords in requests not in trigger list
    weight: 2.0
    confidence_boost: 0.2

  workflow_evolution:
    description: Steps in workflow have changed over time
    detection:
      - Tool sequence differs from documented workflow
      - New tools being used for same task
      - Steps added or removed consistently
    weight: 1.5
    confidence_boost: 0.15

  missing_context:
    description: Files referenced in sessions not in skill context
    detection:
      - Read tool calls for files not in skill directory
      - Same external context loaded 3+ times
    weight: 1.0
    confidence_boost: 0.1

  new_use_cases:
    description: Use cases not covered by current examples
    detection:
      - Requests match skill but don't match any example
      - Edge cases being handled manually
    weight: 1.2
    confidence_boost: 0.1

  frequency_change:
    description: Skill usage pattern has shifted significantly
    detection:
      - Workflow usage increased 2x over baseline
      - Certain workflows deprecated (not used in 60 days)
    weight: 0.8
    confidence_boost: 0.05

thresholds:
  min_confidence: 0.5
  deprecation_days: 60
SIGNALS_EOF
```

#### 4.3: Initialize Proposal Cache

```bash
echo '{"proposals": [], "lastAnalysis": null}' > "$PAI_DIR/Skills/SkillSuggester/Data/proposal-cache.json"
```

---

### Step 5: Create Tool Files

#### 5.1: SkillSuggest.ts (Main CLI)

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Tools/SkillSuggest.ts" << 'CLI_EOF'
#!/usr/bin/env bun
/**
 * SkillSuggest.ts - Main CLI for Skill Suggester
 *
 * Usage:
 *   bun run SkillSuggest.ts [options]
 *
 * Options:
 *   --days N        Analysis depth in days (default: 30)
 *   --type TYPE     new-skills | updates | all (default: all)
 *   --format FMT    markdown | json (default: markdown)
 *   --min-score N   Minimum score threshold (default: 5.0)
 *   --verbose       Show detailed analysis steps
 */

import { HistoryMiner } from './HistoryMiner';
import { PatternClustering } from './PatternClustering';
import { GapDetector } from './GapDetector';
import { UpdateAnalyzer } from './UpdateAnalyzer';
import { ProposalFormatter } from './ProposalFormatter';
import { existsSync, readFileSync, writeFileSync } from 'fs';
import { join } from 'path';

interface CLIOptions {
  days: number;
  type: 'new-skills' | 'updates' | 'all';
  format: 'markdown' | 'json';
  minScore: number;
  verbose: boolean;
}

function parseArgs(): CLIOptions {
  const args = process.argv.slice(2);
  const options: CLIOptions = {
    days: 30,
    type: 'all',
    format: 'markdown',
    minScore: 5.0,
    verbose: false
  };

  for (let i = 0; i < args.length; i++) {
    switch (args[i]) {
      case '--days':
        options.days = parseInt(args[++i], 10) || 30;
        break;
      case '--type':
        options.type = args[++i] as CLIOptions['type'];
        break;
      case '--format':
        options.format = args[++i] as CLIOptions['format'];
        break;
      case '--min-score':
        options.minScore = parseFloat(args[++i]) || 5.0;
        break;
      case '--verbose':
        options.verbose = true;
        break;
    }
  }

  return options;
}

async function main() {
  const options = parseArgs();
  const paiDir = process.env.PAI_DIR || `${process.env.HOME}/.config/pai`;

  console.log(`\n📊 Skill Suggester - Analyzing last ${options.days} days\n`);

  // Check for history data
  const historyDir = join(paiDir, 'history', 'raw-outputs');
  if (!existsSync(historyDir)) {
    console.log('❌ No history data found.');
    console.log('   The history system needs to capture some sessions first.');
    console.log('   Use the system normally, then run this again.\n');
    process.exit(0);
  }

  // Step 1: Mine history
  if (options.verbose) console.log('Step 1: Mining history...');
  const miner = new HistoryMiner(paiDir);
  const events = await miner.mine(options.days);

  if (events.length === 0) {
    console.log('❌ No events found in the specified time range.');
    console.log(`   Try increasing --days (current: ${options.days})\n`);
    process.exit(0);
  }

  console.log(`   Processed ${events.length} events`);

  // Step 2: Cluster patterns
  if (options.verbose) console.log('Step 2: Clustering patterns...');
  const clusterer = new PatternClustering();
  const patterns = clusterer.cluster(events);
  console.log(`   Found ${patterns.length} distinct patterns`);

  // Step 3: Load skill index
  if (options.verbose) console.log('Step 3: Loading skill index...');
  const skillIndexPath = join(paiDir, 'Skills', 'skill-index.json');
  let skillIndex = { skills: {} };
  if (existsSync(skillIndexPath)) {
    skillIndex = JSON.parse(readFileSync(skillIndexPath, 'utf-8'));
  } else {
    console.log('   ⚠️ No skill-index.json found - all patterns will be treated as gaps');
  }

  // Step 4: Detect gaps (new skill opportunities)
  if (options.verbose) console.log('Step 4: Detecting gaps...');
  const gapDetector = new GapDetector();
  const gaps = gapDetector.detect(patterns, skillIndex, options.minScore);

  // Step 5: Analyze update opportunities
  if (options.verbose) console.log('Step 5: Analyzing updates...');
  const updateAnalyzer = new UpdateAnalyzer(paiDir);
  const updates = await updateAnalyzer.analyze(patterns, skillIndex);

  // Step 6: Format and output
  const formatter = new ProposalFormatter();

  if (options.format === 'json') {
    console.log(JSON.stringify({ gaps, updates }, null, 2));
  } else {
    // Summary output
    const summary = formatter.formatSummary(gaps, updates, options.days);
    console.log(summary);
  }

  // Cache proposals for later approval
  const cachePath = join(paiDir, 'Skills', 'SkillSuggester', 'Data', 'proposal-cache.json');
  const cache = {
    proposals: { newSkills: gaps, updates },
    lastAnalysis: new Date().toISOString(),
    options
  };
  writeFileSync(cachePath, JSON.stringify(cache, null, 2));

  if (options.verbose) {
    console.log(`\n✅ Proposals cached at: ${cachePath}`);
  }
}

main().catch(err => {
  console.error('Error:', err.message);
  process.exit(1);
});
CLI_EOF
```

#### 5.2: HistoryMiner.ts

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Tools/HistoryMiner.ts" << 'MINER_EOF'
/**
 * HistoryMiner.ts - Parse JSONL history files for pattern extraction
 */

import { existsSync, readdirSync, readFileSync } from 'fs';
import { join } from 'path';

export interface HistoryEvent {
  source_app: string;
  session_id: string;
  hook_event_type: string;
  tool_name?: string;
  tool_input?: any;
  tool_output?: string;
  timestamp: number;
  timestamp_local: string;
  payload?: any;
}

export interface ExtractedRequest {
  text: string;
  keywords: string[];
  sessionId: string;
  timestamp: number;
  toolSequence: string[];
}

export class HistoryMiner {
  private paiDir: string;
  private maxEvents = 50000;
  private analyzedEventTypes = ['UserPromptSubmit', 'PostToolUse'];

  constructor(paiDir: string) {
    this.paiDir = paiDir;
  }

  async mine(days: number): Promise<ExtractedRequest[]> {
    const rawOutputsDir = join(this.paiDir, 'history', 'raw-outputs');
    if (!existsSync(rawOutputsDir)) {
      return [];
    }

    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);
    const cutoffTimestamp = cutoffDate.getTime();

    const requests: ExtractedRequest[] = [];
    const sessionToolSequences: Map<string, string[]> = new Map();
    let totalProcessed = 0;

    // Get all month directories
    const monthDirs = readdirSync(rawOutputsDir)
      .filter(d => /^\d{4}-\d{2}$/.test(d))
      .sort()
      .reverse(); // Most recent first

    for (const monthDir of monthDirs) {
      const monthPath = join(rawOutputsDir, monthDir);
      const files = readdirSync(monthPath)
        .filter(f => f.endsWith('.jsonl'))
        .sort()
        .reverse();

      for (const file of files) {
        if (totalProcessed >= this.maxEvents) break;

        const filePath = join(monthPath, file);
        const content = readFileSync(filePath, 'utf-8');
        const lines = content.trim().split('\n').filter(l => l);

        for (const line of lines) {
          if (totalProcessed >= this.maxEvents) break;

          try {
            const event: HistoryEvent = JSON.parse(line);

            // Skip if before cutoff
            if (event.timestamp < cutoffTimestamp) continue;

            // Skip if not an analyzed event type
            if (!this.analyzedEventTypes.includes(event.hook_event_type)) continue;

            totalProcessed++;

            // Extract from UserPromptSubmit
            if (event.hook_event_type === 'UserPromptSubmit') {
              const prompt = event.payload?.prompt || '';
              if (prompt.length > 10) {
                requests.push({
                  text: prompt,
                  keywords: this.extractKeywords(prompt),
                  sessionId: event.session_id,
                  timestamp: event.timestamp,
                  toolSequence: []
                });
              }
            }

            // Track tool sequences for workflow detection
            if (event.hook_event_type === 'PostToolUse' && event.tool_name) {
              const sessionId = event.session_id;
              if (!sessionToolSequences.has(sessionId)) {
                sessionToolSequences.set(sessionId, []);
              }
              sessionToolSequences.get(sessionId)!.push(event.tool_name);
            }
          } catch (e) {
            // Skip malformed lines
          }
        }
      }
    }

    // Attach tool sequences to requests from same session
    for (const request of requests) {
      const sequence = sessionToolSequences.get(request.sessionId) || [];
      request.toolSequence = sequence.slice(0, 10); // Cap at 10 tools
    }

    return requests;
  }

  private extractKeywords(text: string): string[] {
    // Normalize and tokenize
    const normalized = text.toLowerCase()
      .replace(/[^\w\s-]/g, ' ')
      .replace(/\s+/g, ' ')
      .trim();

    // Extract meaningful words (3+ chars, not common stopwords)
    const stopwords = new Set([
      'the', 'and', 'for', 'that', 'this', 'with', 'from', 'are', 'was',
      'have', 'has', 'had', 'been', 'will', 'would', 'could', 'should',
      'can', 'may', 'might', 'must', 'shall', 'need', 'want', 'like',
      'just', 'also', 'very', 'really', 'please', 'help', 'make', 'use'
    ]);

    return normalized.split(' ')
      .filter(word => word.length >= 3 && !stopwords.has(word))
      .slice(0, 20); // Cap keywords
  }
}
MINER_EOF
```

#### 5.3: PatternClustering.ts

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Tools/PatternClustering.ts" << 'CLUSTER_EOF'
/**
 * PatternClustering.ts - Group similar requests into patterns
 */

import type { ExtractedRequest } from './HistoryMiner';

export interface PatternCluster {
  id: string;
  canonicalForm: string;
  keywords: string[];
  instances: ExtractedRequest[];
  frequency: number;
  sessionCount: number;
  toolSequence: string[];
  score: number;
}

export class PatternClustering {
  private similarityThreshold = 0.4;

  cluster(requests: ExtractedRequest[]): PatternCluster[] {
    const clusters: Map<string, PatternCluster> = new Map();

    for (const request of requests) {
      if (request.keywords.length === 0) continue;

      // Find best matching cluster
      let bestMatch: { cluster: PatternCluster; similarity: number } | null = null;

      for (const cluster of clusters.values()) {
        const similarity = this.computeSimilarity(request.keywords, cluster.keywords);
        if (similarity >= this.similarityThreshold) {
          if (!bestMatch || similarity > bestMatch.similarity) {
            bestMatch = { cluster, similarity };
          }
        }
      }

      if (bestMatch) {
        // Add to existing cluster
        bestMatch.cluster.instances.push(request);
        bestMatch.cluster.frequency++;
        // Update keywords (union)
        const keywordSet = new Set([...bestMatch.cluster.keywords, ...request.keywords]);
        bestMatch.cluster.keywords = Array.from(keywordSet).slice(0, 15);
      } else {
        // Create new cluster
        const id = this.generateId(request.keywords);
        clusters.set(id, {
          id,
          canonicalForm: request.text.slice(0, 100),
          keywords: request.keywords.slice(0, 15),
          instances: [request],
          frequency: 1,
          sessionCount: 1,
          toolSequence: request.toolSequence,
          score: 0
        });
      }
    }

    // Calculate session counts and scores
    for (const cluster of clusters.values()) {
      const uniqueSessions = new Set(cluster.instances.map(i => i.sessionId));
      cluster.sessionCount = uniqueSessions.size;

      // Compute most common tool sequence
      const sequenceCounts = new Map<string, number>();
      for (const instance of cluster.instances) {
        const seqKey = instance.toolSequence.slice(0, 5).join('→');
        if (seqKey) {
          sequenceCounts.set(seqKey, (sequenceCounts.get(seqKey) || 0) + 1);
        }
      }
      if (sequenceCounts.size > 0) {
        const topSequence = [...sequenceCounts.entries()]
          .sort((a, b) => b[1] - a[1])[0][0];
        cluster.toolSequence = topSequence.split('→');
      }

      // Score calculation
      cluster.score = this.computeScore(cluster);
    }

    // Filter and sort by score
    return Array.from(clusters.values())
      .filter(c => c.frequency >= 2) // At least 2 occurrences
      .sort((a, b) => b.score - a.score);
  }

  private computeSimilarity(keywords1: string[], keywords2: string[]): number {
    // Jaccard similarity
    const set1 = new Set(keywords1);
    const set2 = new Set(keywords2);

    const intersection = new Set([...set1].filter(x => set2.has(x)));
    const union = new Set([...set1, ...set2]);

    if (union.size === 0) return 0;
    return intersection.size / union.size;
  }

  private generateId(keywords: string[]): string {
    return keywords.slice(0, 3).join('-').toLowerCase().replace(/[^a-z0-9-]/g, '');
  }

  private computeScore(cluster: PatternCluster): number {
    let score = 0;

    // Base score from frequency
    score += cluster.frequency * 1.0;

    // Bonus for multiple sessions
    score += cluster.sessionCount * 0.5;

    // Bonus for tool sequences (indicates workflow)
    if (cluster.toolSequence.length >= 3) {
      score += 1.5;
    }

    // Bonus for specific keywords (domain indicators)
    const domainKeywords = ['docker', 'api', 'database', 'deploy', 'test', 'build', 'config', 'setup'];
    const hasDomainKeyword = cluster.keywords.some(k =>
      domainKeywords.some(dk => k.includes(dk))
    );
    if (hasDomainKeyword) {
      score += 1.0;
    }

    return Math.round(score * 10) / 10;
  }
}
CLUSTER_EOF
```

#### 5.4: GapDetector.ts

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Tools/GapDetector.ts" << 'GAP_EOF'
/**
 * GapDetector.ts - Compare patterns against existing skill coverage
 */

import type { PatternCluster } from './PatternClustering';

export interface SkillIndex {
  skills: Record<string, SkillEntry>;
}

export interface SkillEntry {
  name: string;
  path: string;
  fullDescription: string;
  triggers: string[];
  workflows: string[];
  tier: 'always' | 'deferred';
}

export interface SkillGap {
  pattern: PatternCluster;
  matchedSkills: SkillMatch[];
  gapType: 'no_match' | 'partial_match';
  confidence: number;
  proposedName: string;
  proposedTriggers: string[];
  proposedWorkflows: string[];
  evidence: string[];
}

export interface SkillMatch {
  skillName: string;
  triggerScore: number;
  descriptionScore: number;
  totalScore: number;
}

export class GapDetector {
  private triggerMatchThreshold = 30;

  detect(patterns: PatternCluster[], skillIndex: SkillIndex, minScore: number): SkillGap[] {
    const gaps: SkillGap[] = [];

    for (const pattern of patterns) {
      if (pattern.score < minScore) continue;

      const matches = this.scoreAgainstSkills(pattern, skillIndex);
      const bestMatch = matches[0];

      // No good match - this is a gap
      if (!bestMatch || bestMatch.totalScore < this.triggerMatchThreshold) {
        const confidence = this.calculateConfidence(pattern, matches);

        gaps.push({
          pattern,
          matchedSkills: matches.slice(0, 3),
          gapType: 'no_match',
          confidence,
          proposedName: this.generateSkillName(pattern),
          proposedTriggers: this.generateTriggers(pattern),
          proposedWorkflows: this.generateWorkflows(pattern),
          evidence: this.extractEvidence(pattern)
        });
      }
    }

    return gaps.sort((a, b) => b.confidence - a.confidence);
  }

  private scoreAgainstSkills(pattern: PatternCluster, index: SkillIndex): SkillMatch[] {
    const matches: SkillMatch[] = [];

    for (const [key, skill] of Object.entries(index.skills)) {
      const triggerScore = this.scoreTriggerMatch(pattern.keywords, skill.triggers);
      const descriptionScore = this.scoreDescriptionMatch(pattern.keywords, skill.fullDescription);
      const totalScore = (triggerScore * 0.6) + (descriptionScore * 0.4);

      if (totalScore > 0) {
        matches.push({
          skillName: skill.name,
          triggerScore,
          descriptionScore,
          totalScore
        });
      }
    }

    return matches.sort((a, b) => b.totalScore - a.totalScore);
  }

  private scoreTriggerMatch(keywords: string[], triggers: string[]): number {
    if (!triggers || triggers.length === 0) return 0;

    let matchCount = 0;
    for (const keyword of keywords) {
      for (const trigger of triggers) {
        if (trigger.includes(keyword) || keyword.includes(trigger)) {
          matchCount++;
          break;
        }
      }
    }

    return Math.min(100, (matchCount / Math.max(keywords.length, 1)) * 100);
  }

  private scoreDescriptionMatch(keywords: string[], description: string): number {
    if (!description) return 0;

    const descLower = description.toLowerCase();
    let matchCount = 0;

    for (const keyword of keywords) {
      if (descLower.includes(keyword)) {
        matchCount++;
      }
    }

    return Math.min(100, (matchCount / Math.max(keywords.length, 1)) * 100);
  }

  private calculateConfidence(pattern: PatternCluster, matches: SkillMatch[]): number {
    // Base confidence from pattern strength
    let confidence = Math.min(0.95, pattern.score / 15);

    // Boost if pattern is clearly different from existing skills
    if (matches.length === 0 || matches[0].totalScore < 10) {
      confidence += 0.1;
    }

    // Boost for multiple sessions
    if (pattern.sessionCount >= 3) {
      confidence += 0.05;
    }

    return Math.min(0.95, Math.round(confidence * 100) / 100);
  }

  private generateSkillName(pattern: PatternCluster): string {
    // Use top keywords to form a TitleCase name
    const topKeywords = pattern.keywords.slice(0, 2);
    const name = topKeywords
      .map(k => k.charAt(0).toUpperCase() + k.slice(1))
      .join('');

    return name || 'CustomWorkflow';
  }

  private generateTriggers(pattern: PatternCluster): string[] {
    return pattern.keywords.slice(0, 8);
  }

  private generateWorkflows(pattern: PatternCluster): string[] {
    const workflows: string[] = [];

    // If there's a clear tool sequence, suggest a workflow
    if (pattern.toolSequence.length >= 2) {
      const workflowName = pattern.keywords[0]
        ? pattern.keywords[0].charAt(0).toUpperCase() + pattern.keywords[0].slice(1) + 'Process'
        : 'MainWorkflow';
      workflows.push(workflowName);
    }

    return workflows;
  }

  private extractEvidence(pattern: PatternCluster): string[] {
    return pattern.instances
      .slice(0, 5)
      .map(i => {
        const date = new Date(i.timestamp).toISOString().split('T')[0];
        const text = i.text.slice(0, 60) + (i.text.length > 60 ? '...' : '');
        return `${date}: "${text}"`;
      });
  }
}
GAP_EOF
```

#### 5.5: UpdateAnalyzer.ts

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Tools/UpdateAnalyzer.ts" << 'UPDATE_EOF'
/**
 * UpdateAnalyzer.ts - Detect skills that need updates
 */

import { existsSync, readFileSync } from 'fs';
import { join } from 'path';
import type { PatternCluster } from './PatternClustering';
import type { SkillIndex, SkillEntry } from './GapDetector';

export interface SkillUpdate {
  skillName: string;
  skillPath: string;
  updateType: 'triggers' | 'workflow' | 'context' | 'examples';
  current: string;
  proposed: string;
  evidence: UpdateEvidence[];
  confidence: number;
}

export interface UpdateEvidence {
  date: string;
  request: string;
  mismatch: string;
}

export class UpdateAnalyzer {
  private paiDir: string;
  private minConfidence = 0.5;

  constructor(paiDir: string) {
    this.paiDir = paiDir;
  }

  async analyze(patterns: PatternCluster[], skillIndex: SkillIndex): Promise<SkillUpdate[]> {
    const updates: SkillUpdate[] = [];

    for (const [key, skill] of Object.entries(skillIndex.skills)) {
      // Find patterns that partially match this skill
      const relatedPatterns = patterns.filter(p =>
        this.isRelatedToSkill(p, skill)
      );

      if (relatedPatterns.length === 0) continue;

      // Check for trigger mismatches
      const triggerUpdates = this.detectTriggerMismatches(skill, relatedPatterns);
      updates.push(...triggerUpdates);

      // Check for new use cases
      const useCaseUpdates = this.detectNewUseCases(skill, relatedPatterns);
      updates.push(...useCaseUpdates);
    }

    return updates
      .filter(u => u.confidence >= this.minConfidence)
      .sort((a, b) => b.confidence - a.confidence);
  }

  private isRelatedToSkill(pattern: PatternCluster, skill: SkillEntry): boolean {
    // Check if pattern keywords overlap with skill triggers or description
    const skillWords = [
      ...skill.triggers,
      ...skill.fullDescription.toLowerCase().split(/\s+/)
    ];

    let matchCount = 0;
    for (const keyword of pattern.keywords) {
      if (skillWords.some(sw => sw.includes(keyword) || keyword.includes(sw))) {
        matchCount++;
      }
    }

    // At least 2 keyword matches
    return matchCount >= 2;
  }

  private detectTriggerMismatches(skill: SkillEntry, patterns: PatternCluster[]): SkillUpdate[] {
    const updates: SkillUpdate[] = [];
    const missingTriggers = new Set<string>();
    const evidence: UpdateEvidence[] = [];

    for (const pattern of patterns) {
      for (const keyword of pattern.keywords) {
        // Check if keyword should be a trigger but isn't
        const isTrigger = skill.triggers.some(t =>
          t.includes(keyword) || keyword.includes(t)
        );

        if (!isTrigger && pattern.frequency >= 3) {
          missingTriggers.add(keyword);

          // Add evidence
          const instance = pattern.instances[0];
          if (instance) {
            evidence.push({
              date: new Date(instance.timestamp).toISOString().split('T')[0],
              request: instance.text.slice(0, 50),
              mismatch: `"${keyword}" not in triggers`
            });
          }
        }
      }
    }

    if (missingTriggers.size > 0) {
      const currentTriggers = skill.triggers.join(', ');
      const proposedTriggers = [...skill.triggers, ...missingTriggers].join(', ');

      updates.push({
        skillName: skill.name,
        skillPath: skill.path,
        updateType: 'triggers',
        current: currentTriggers,
        proposed: proposedTriggers,
        evidence: evidence.slice(0, 5),
        confidence: Math.min(0.95, 0.6 + (missingTriggers.size * 0.1))
      });
    }

    return updates;
  }

  private detectNewUseCases(skill: SkillEntry, patterns: PatternCluster[]): SkillUpdate[] {
    const updates: SkillUpdate[] = [];

    // Read skill file to check examples
    const skillPath = join(this.paiDir, 'Skills', skill.path);
    if (!existsSync(skillPath)) return updates;

    const skillContent = readFileSync(skillPath, 'utf-8');
    const hasExamplesSection = skillContent.includes('## Examples');

    if (!hasExamplesSection) {
      // Skill lacks examples - suggest adding from patterns
      const evidence: UpdateEvidence[] = patterns.slice(0, 3).flatMap(p =>
        p.instances.slice(0, 2).map(i => ({
          date: new Date(i.timestamp).toISOString().split('T')[0],
          request: i.text.slice(0, 50),
          mismatch: 'Could be an example'
        }))
      );

      if (evidence.length >= 2) {
        updates.push({
          skillName: skill.name,
          skillPath: skill.path,
          updateType: 'examples',
          current: 'No examples section',
          proposed: `Add ${evidence.length} examples from recent usage`,
          evidence,
          confidence: 0.6
        });
      }
    }

    return updates;
  }
}
UPDATE_EOF
```

#### 5.6: ProposalFormatter.ts

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Tools/ProposalFormatter.ts" << 'FORMAT_EOF'
/**
 * ProposalFormatter.ts - Generate summary and detailed proposal output
 */

import type { SkillGap } from './GapDetector';
import type { SkillUpdate } from './UpdateAnalyzer';

export class ProposalFormatter {
  formatSummary(gaps: SkillGap[], updates: SkillUpdate[], days: number): string {
    const lines: string[] = [];

    lines.push(`📊 Skill Analysis Complete (last ${days} days)\n`);

    // New skill opportunities
    lines.push(`**New Skill Opportunities:** ${gaps.length} found`);
    if (gaps.length > 0) {
      for (const gap of gaps.slice(0, 5)) {
        const confidence = Math.round(gap.confidence * 100);
        lines.push(`  ${gaps.indexOf(gap) + 1}. ${gap.proposedName} (${confidence}%) - ${gap.pattern.frequency} occurrences across ${gap.pattern.sessionCount} sessions`);
      }
      if (gaps.length > 5) {
        lines.push(`  ... and ${gaps.length - 5} more`);
      }
    }
    lines.push('');

    // Skill updates
    lines.push(`**Existing Skill Updates:** ${updates.length} suggested`);
    if (updates.length > 0) {
      for (const update of updates.slice(0, 5)) {
        const confidence = Math.round(update.confidence * 100);
        lines.push(`  ${updates.indexOf(update) + 1}. ${update.skillName} - ${update.updateType} (${confidence}%)`);
      }
      if (updates.length > 5) {
        lines.push(`  ... and ${updates.length - 5} more`);
      }
    }
    lines.push('');

    // Instructions
    if (gaps.length > 0 || updates.length > 0) {
      lines.push('Say "show me [name]" for details, or "create [name]" to proceed.');
    } else {
      lines.push('No skill opportunities or updates detected.');
      lines.push('Try analyzing a longer time period with --days N');
    }

    return lines.join('\n');
  }

  formatGapDetail(gap: SkillGap): string {
    const lines: string[] = [];
    const confidence = Math.round(gap.confidence * 100);

    lines.push(`## 📦 ${gap.proposedName}`);
    lines.push(`**Confidence:** ${confidence}% | **Evidence:** ${gap.pattern.frequency} occurrences across ${gap.pattern.sessionCount} sessions`);
    lines.push('');
    lines.push(`**Proposed Triggers:** ${gap.proposedTriggers.join(', ')}`);
    lines.push('');

    if (gap.proposedWorkflows.length > 0) {
      lines.push('**Suggested Workflows:**');
      for (const wf of gap.proposedWorkflows) {
        lines.push(`- ${wf}`);
      }
      lines.push('');
    }

    if (gap.pattern.toolSequence.length > 0) {
      lines.push(`**Tool Sequence:** ${gap.pattern.toolSequence.join(' → ')}`);
      lines.push('');
    }

    lines.push('**Recent Evidence:**');
    for (const ev of gap.evidence) {
      lines.push(`- ${ev}`);
    }
    lines.push('');
    lines.push(`Say "create ${gap.proposedName}" to generate this skill.`);

    return lines.join('\n');
  }

  formatUpdateDetail(update: SkillUpdate): string {
    const lines: string[] = [];
    const confidence = Math.round(update.confidence * 100);

    lines.push(`## 🔄 ${update.skillName} Update`);
    lines.push(`**Type:** ${update.updateType} | **Confidence:** ${confidence}%`);
    lines.push('');

    // Side-by-side format
    lines.push('```');
    lines.push('┌─────────────────────────────────┬─────────────────────────────────┐');
    lines.push('│ CURRENT                         │ PROPOSED                        │');
    lines.push('├─────────────────────────────────┼─────────────────────────────────┤');

    const currentLines = this.wrapText(update.current, 31);
    const proposedLines = this.wrapText(update.proposed, 31);
    const maxLines = Math.max(currentLines.length, proposedLines.length);

    for (let i = 0; i < maxLines; i++) {
      const curr = (currentLines[i] || '').padEnd(31);
      const prop = (proposedLines[i] || '').padEnd(31);
      lines.push(`│ ${curr} │ ${prop} │`);
    }

    lines.push('└─────────────────────────────────┴─────────────────────────────────┘');
    lines.push('```');
    lines.push('');

    lines.push('**Evidence:**');
    for (const ev of update.evidence.slice(0, 3)) {
      lines.push(`- ${ev.date}: "${ev.request}" - ${ev.mismatch}`);
    }
    lines.push('');
    lines.push(`Say "update ${update.skillName}" to apply these changes.`);

    return lines.join('\n');
  }

  private wrapText(text: string, width: number): string[] {
    const words = text.split(' ');
    const lines: string[] = [];
    let currentLine = '';

    for (const word of words) {
      if (currentLine.length + word.length + 1 <= width) {
        currentLine += (currentLine ? ' ' : '') + word;
      } else {
        if (currentLine) lines.push(currentLine);
        currentLine = word.slice(0, width);
      }
    }
    if (currentLine) lines.push(currentLine);

    return lines.length > 0 ? lines : [''];
  }
}
FORMAT_EOF
```

---

### Step 6: Create Workflows

#### 6.1: AnalyzeHistory.md

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Workflows/AnalyzeHistory.md" << 'ANALYZE_EOF'
# AnalyzeHistory Workflow

> **Trigger:** "suggest skills", "analyze patterns", "what skills should I create"

## Purpose

Analyze PAI history to discover skill opportunities and propose updates.

## Steps

### Step 1: Run Analysis

Execute the SkillSuggest CLI tool:

```bash
bun run $PAI_DIR/Skills/SkillSuggester/Tools/SkillSuggest.ts --days 30 --type all
```

Options:
- `--days N` to change analysis depth (default: 30)
- `--type new-skills` for only new skill suggestions
- `--type updates` for only update suggestions

### Step 2: Present Summary

Display the summary output to the user:
- Number of new skill opportunities found
- Number of update suggestions
- Brief description of each

### Step 3: Await User Direction

The user can:
- Say "show me [name]" to see detailed proposal
- Say "create [name]" to proceed with creation
- Say "update [name]" to apply an update
- Ask for longer analysis period

### Step 4: Load Cached Proposals

If user requests details, read from:
```
$PAI_DIR/Skills/SkillSuggester/Data/proposal-cache.json
```

And format the appropriate detail view (gap or update).

## Example Flow

```
User: "Suggest some skills for me"

1. Run SkillSuggest.ts
2. Display:
   📊 Skill Analysis Complete (last 30 days)

   **New Skill Opportunities:** 2 found
     1. DockerWorkflow (87%) - 12 occurrences
     2. DatabaseMigrations (72%) - 8 occurrences

   Say "show me [name]" for details...

3. User: "Show me DockerWorkflow"
4. Display detailed proposal with evidence
5. User: "Create DockerWorkflow"
6. Route to ApplyProposal workflow
```
ANALYZE_EOF
```

#### 6.2: ApplyProposal.md

```bash
cat > "$PAI_DIR/Skills/SkillSuggester/Workflows/ApplyProposal.md" << 'APPLY_EOF'
# ApplyProposal Workflow

> **Trigger:** "create [SkillName]", "update [SkillName]"

## Purpose

Execute an approved proposal by delegating to CreateSkill or updating an existing skill.

## Steps

### Step 1: Load Proposal from Cache

Read the proposal from:
```
$PAI_DIR/Skills/SkillSuggester/Data/proposal-cache.json
```

Find the matching proposal by name.

### Step 2: For New Skills

Invoke the CreateSkill workflow with the generated template:

1. Create the skill directory structure
2. Generate SKILL.md with:
   - Frontmatter: name, description with USE WHEN triggers
   - Workflow routing table
   - Examples section
3. Create placeholder workflow files
4. Create empty Tools/ directory

```bash
# After skill is created, refresh the index
bun run $PAI_DIR/Tools/GenerateSkillIndex.ts
```

### Step 3: For Updates

Read the existing SKILL.md and apply changes:

1. For trigger updates: Modify the description USE WHEN clause
2. For workflow updates: Add new workflow file and routing entry
3. For example updates: Add new examples to Examples section

```bash
# After update, refresh the index
bun run $PAI_DIR/Tools/GenerateSkillIndex.ts
```

### Step 4: Confirm Completion

Output confirmation:
```
Created: $PAI_DIR/Skills/[SkillName]/
- SKILL.md with triggers
- Workflows/[WorkflowName].md
- Tools/ directory

Skill index updated. [SkillName] is now available.
```

### Step 5: Clear from Cache

Remove the applied proposal from proposal-cache.json.

## Example

```
User: "Create DockerWorkflow"

1. Load DockerWorkflow proposal from cache
2. Create directory: $PAI_DIR/Skills/DockerWorkflow/
3. Write SKILL.md:
   ---
   name: DockerWorkflow
   description: Docker container workflows. USE WHEN docker, container, dockerfile, compose.
   ---
4. Create Workflows/DockerProcess.md
5. Run GenerateSkillIndex.ts
6. Output confirmation
7. Remove proposal from cache
```
APPLY_EOF
```

---

### Step 7: Verification

```bash
# 1. Check all files exist
echo "Checking file structure..."
ls -la "$PAI_DIR/Skills/SkillSuggester/"
ls -la "$PAI_DIR/Skills/SkillSuggester/Tools/"
ls -la "$PAI_DIR/Skills/SkillSuggester/Workflows/"
ls -la "$PAI_DIR/Skills/SkillSuggester/Data/"

# 2. Verify tool runs
echo ""
echo "Testing CLI tool..."
bun run "$PAI_DIR/Skills/SkillSuggester/Tools/SkillSuggest.ts" --days 7

# 3. Check skill-index.json can be regenerated
echo ""
echo "Regenerating skill index..."
bun run "$PAI_DIR/Tools/GenerateSkillIndex.ts"

# 4. Search for new skill
echo ""
echo "Searching for SkillSuggester..."
bun run "$PAI_DIR/Tools/SkillSearch.ts" "suggest skills"
```

---

## Invocation Scenarios

| Trigger | What Happens |
|---------|-------------|
| "Suggest some skills for me" | Runs AnalyzeHistory workflow, shows summary |
| "What skills should I create?" | Same as above |
| "Analyze my patterns" | Same as above |
| "/skill-suggest" | Direct CLI invocation |
| "Show me [SkillName]" | Loads cached proposal, shows details |
| "Create [SkillName]" | Runs ApplyProposal workflow for new skill |
| "Update [SkillName]" | Runs ApplyProposal workflow for update |

---

## Example Usage

### Example 1: Basic Suggestion Flow

```
User: "Suggest some skills for me"

SkillSuggester:
📊 Skill Analysis Complete (last 30 days)

**New Skill Opportunities:** 2 found
  1. DockerWorkflow (87%) - 12 occurrences across 5 sessions
  2. DatabaseMigrations (72%) - 8 occurrences across 3 sessions

**Existing Skill Updates:** 1 suggested
  1. Art - triggers (82%)

Say "show me [name]" for details, or "create [name]" to proceed.
```

### Example 2: Drill Down

```
User: "Show me DockerWorkflow"

SkillSuggester:
## 📦 DockerWorkflow
**Confidence:** 87% | **Evidence:** 12 occurrences across 5 sessions

**Proposed Triggers:** docker, container, dockerfile, compose, build, image

**Tool Sequence:** Bash → Edit → Bash

**Recent Evidence:**
- 2025-12-28: "build a docker image for the api"
- 2025-12-27: "deploy the container to staging"
- 2025-12-26: "create a dockerfile for this project"

Say "create DockerWorkflow" to generate this skill.
```

### Example 3: Create Skill

```
User: "Create DockerWorkflow"

SkillSuggester:
Creating DockerWorkflow skill...

Created: $PAI_DIR/Skills/DockerWorkflow/
├── SKILL.md (triggers: docker, container, dockerfile, compose)
├── Workflows/
│   └── DockerProcess.md
└── Tools/

Skill index updated. DockerWorkflow is now available.

Try: "I need to build a docker image" to invoke your new skill.
```

---

## Configuration

**Environment variables:**

**Option 1: `.env` file** (recommended):
```bash
# $PAI_DIR/.env
PAI_DIR="$HOME/.config/pai"
```

**Option 2: Shell profile**:
```bash
# Add to ~/.zshrc or ~/.bashrc
export PAI_DIR="$HOME/.config/pai"
```

**Analysis thresholds:**

Edit `$PAI_DIR/Skills/SkillSuggester/AnalysisConfig.md` to adjust:
- `minOccurrences`: How many times a pattern must appear (default: 3)
- `minSessions`: Minimum unique sessions (default: 2)
- `minScore`: Minimum worthiness score (default: 5.0)
- `defaultDays`: Default analysis period (default: 30)

---

## Customization

### Recommended Customization

**Adjust thresholds for your usage volume:**

If you're a high-volume user (10+ sessions/day):
- Increase `minOccurrences` to 5
- Increase `minSessions` to 3

If you're a low-volume user (<1 session/day):
- Decrease `minOccurrences` to 2
- Increase `defaultDays` to 60

Edit `$PAI_DIR/Skills/SkillSuggester/Data/PatternHeuristics.yaml` to customize.

### Optional Customization

| Customization | File | Impact |
|--------------|------|--------|
| Add domain-specific keywords | `PatternHeuristics.yaml` | Better pattern recognition |
| Change update signals | `UpdateSignals.yaml` | Different update detection |
| Modify similarity threshold | `PatternClustering.ts` | Tighter/looser clustering |

---

## Credits

- **Original concept**: Daniel Miessler - developed as part of Kai personal AI infrastructure
- **Architecture design**: Built on top of kai-history-system and kai-core-install foundations

---

## Related Work

*None specified - maintainer to provide if applicable.*

---

## Works Well With

- **kai-history-system**: Required - provides the raw-outputs data for analysis
- **kai-core-install**: Required - provides CreateSkill workflow and skill-index.json

---

## Recommended

- **kai-input-enhancer**: Complements by classifying requests in real-time

---

## Relationships

### Parent Of
*None specified.*

### Child Of
- kai-core-install (uses CreateSkill workflow)
- kai-history-system (uses raw-outputs data)

### Sibling Of
*None specified.*

### Part Of Collection
- Kai Bundle

---

## Changelog

### 1.0.0 - 2025-12-30
- Initial release
- History mining with chunked processing
- Pattern clustering with keyword similarity
- Gap detection against skill-index.json
- Update detection for trigger mismatches
- Summary-first presentation with drill-down
- Integration with CreateSkill workflow
