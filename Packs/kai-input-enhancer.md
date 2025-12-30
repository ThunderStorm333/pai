---
name: Kai Input Enhancer
pack-id: danielmiessler-kai-input-enhancer-core-v1.0.0
version: 1.0.0
author: danielmiessler
description: Two-part input enhancement system - Hook classifies and recommends; Skill handles preview, editing, questions, and agent composition for complex tasks
type: feature
purpose-type: [productivity, automation, development]
platform: claude-code
dependencies:
  - danielmiessler-kai-hook-system-core-v1.0.0
  - danielmiessler-kai-core-install-core-v1.0.0
  - danielmiessler-kai-prompting-skill-core-v1.0.0
keywords: [input, enhancement, prompting, classification, chain-of-thought, agents, multi-agent, preview, clarification, meta-prompting, automation]
---

<p align="center">
  <img src="icons/kai-input-enhancer.png" alt="Kai Input Enhancer" width="256">
</p>

# Kai Input Enhancer (kai-input-enhancer)

> Two-part input enhancement system that automatically classifies, improves, and structures user requests for optimal AI understanding

## Installation Prompt

You are receiving a PAI Pack - a modular upgrade for AI agent systems.

**What is PAI?** See: [PAI Project Overview](../README.md#what-is-pai)

**What is a Pack?** See: [Pack System](../README.md#pack-system)

This Pack adds automatic input enhancement to your AI infrastructure. The Kai Input Enhancer is a two-part system that improves every request you make:

- **Automatic Classification**: Categorize requests into 11 types (debug, feature, explain, etc.)
- **Chain-of-Thought Injection**: Add structured thinking for complex tasks
- **Plan Mode Awareness**: Shift to research-focused enhancement during planning
- **Preview & Edit**: See enhancements before they're applied
- **Multi-Agent Composition**: Propose specialized agent teams for complex tasks
- **Clarification Questions**: Ask targeted questions when input is ambiguous

**Core principle:** Better inputs = better outputs. Automatically.

The hook fires on every input, classifies it, and recommends enhancements. For complex tasks, Claude invokes the InputEnhancer skill for interactive preview, editing, and agent composition.

Please follow the installation instructions below to integrate this Pack into your infrastructure.

---

## What's Included

| Component | File | Purpose |
|-----------|------|---------|
| Main hook | `hooks/input-enhancer.ts` | Classify input, detect complexity, output recommendations |
| Types library | `hooks/lib/input-enhancer/types.ts` | Type definitions for classification |
| Quick filter | `hooks/lib/input-enhancer/quick-filter.ts` | Skip short/bypass/command inputs |
| Classifier | `hooks/lib/input-enhancer/classifier.ts` | Classify into 11 request types |
| Complexity detector | `hooks/lib/input-enhancer/complexity-detector.ts` | Detect when agents would help |
| Plan mode detector | `hooks/lib/input-enhancer/plan-mode-detector.ts` | Detect and adjust for plan mode |
| Enhancement generator | `hooks/lib/input-enhancer/enhancement-generator.ts` | Generate CoT enhancements |
| Skill definition | `Skills/InputEnhancer/SKILL.md` | Skill routing and overview |
| Preview workflow | `Skills/InputEnhancer/Workflows/PreviewEnhancement.md` | Preview + edit flow |
| Agent team workflow | `Skills/InputEnhancer/Workflows/ComposeAgentTeam.md` | Agent composition flow |
| Clarification workflow | `Skills/InputEnhancer/Workflows/AskClarification.md` | Question-asking flow |
| CoT templates | `Skills/InputEnhancer/Data/EnhancementTemplates.yaml` | Chain-of-thought by type |
| Agent profiles | `Skills/InputEnhancer/Data/AgentProfiles.yaml` | Pre-defined agent teams |
| Question patterns | `Skills/InputEnhancer/Data/QuestionPatterns.yaml` | Questions by gap type |
| Enhancement tool | `Skills/InputEnhancer/Tools/GenerateEnhancement.ts` | CLI for generating enhancements |

**Summary:**
- **Files created:** 15
- **Hooks registered:** 1 (UserPromptSubmit)
- **Dependencies:** kai-hook-system (required), kai-core-install (required), kai-prompting-skill (required), kai-agents-skill (optional), kai-history-system (optional)

---

## The Concept and/or Problem

AI assistants are only as good as the prompts they receive. But most users don't know:

- **How to structure requests** - "fix the bug" vs "Debug the TypeError in auth.ts by tracing the data flow"
- **What context to provide** - Missing scope, target files, expected behavior
- **When multi-perspective analysis helps** - Security audits, code reviews, research tasks
- **How to think step-by-step** - Chain-of-thought improves complex reasoning

This creates a gap between what users ask and what they need:

**For Simple Requests:**
- "explain this" - Explain what? The whole file? One function? The architecture?
- "fix it" - Fix what? Where? What's the symptom?
- "add a button" - Where? What should it do? What style?

**For Complex Tasks:**
- Security audits benefit from multiple perspectives (adversarial, defensive, compliance)
- Research tasks need exploration, critique, and synthesis
- Architecture decisions need trade-off analysis from different viewpoints

**For Plan Mode:**
- Implementation-focused prompts are wrong during research phase
- Need to shift from "how to build" to "what to understand first"
- Planning requires different chain-of-thought than execution

**The Fundamental Problem:**

Users naturally think in terms of outcomes ("make it work") but AI needs structured inputs with context, constraints, and success criteria. This mismatch causes:

- Repeated clarifications that slow work down
- AI making assumptions that turn out to be wrong
- Complex tasks getting shallow treatment
- Plan mode research turning into premature implementation

Traditional prompt improvement tools require manual intervention. You have to stop, write a better prompt, then submit. This breaks flow and requires expertise most users don't have.

---

## The Solution

The Kai Input Enhancer solves this through a **two-part architecture** that separates automatic analysis from interactive refinement:

**Critical Constraint:** Hooks cannot wait for user input. They fire and exit in milliseconds.

**Solution:** Hook recommends, Skill interacts.

```
User Input
    |
    v
+---------------------------------------------------------------+
|  PART 1: HOOK (input-enhancer.ts)                             |
|  Fires on: UserPromptSubmit                                   |
|  Duration: Milliseconds (fire-and-forget)                     |
|                                                               |
|  1. Quick filter (skip short/commands/bypass)                 |
|  2. Classify (type, confidence, complexity)                   |
|  3. Detect plan mode and adjust strategy                      |
|  4. Detect gaps (missing scope, target, etc.)                 |
|  5. Output recommendation as <system-reminder>                |
|  6. Exit 0 (cannot block or wait)                             |
+---------------------------------------------------------------+
    |
    | Claude sees recommendation in system-reminder
    v
+---------------------------------------------------------------+
|  CLAUDE DECIDES                                               |
|                                                               |
|  Simple + Clear -> Apply light enhancement, proceed           |
|  Complex task -> Invoke InputEnhancer skill                   |
|  Already clear -> Skip enhancement                            |
+---------------------------------------------------------------+
    |
    | if complex/ambiguous
    v
+---------------------------------------------------------------+
|  PART 2: SKILL (Skills/InputEnhancer/SKILL.md)                |
|  Invoked by: Claude based on hook recommendation              |
|  Duration: Interactive (normal chat flow)                     |
|                                                               |
|  Workflows:                                                   |
|  - PreviewEnhancement: Show + edit before applying            |
|  - ComposeAgentTeam: Propose + customize agent team           |
|  - AskClarification: Questions for ambiguous input            |
+---------------------------------------------------------------+
    |
    v
[Enhanced request + optional agents proceed]
```

**Design Principles:**

1. **Zero Friction**: Hook runs automatically, no user action required
2. **Progressive Enhancement**: Simple inputs get light touch, complex get full treatment
3. **User Control**: Preview and edit before applying (via skill)
4. **Plan Mode Aware**: Detects planning phase, shifts to research-focused prompts
5. **Agent Integration**: Compose specialized teams for multi-perspective analysis
6. **Graceful Degradation**: Works without optional dependencies

---

## What Makes This Different

The Input Enhancer's power comes from its **two-part architecture** that separates fast classification from interactive refinement, with **plan mode awareness** that adapts enhancement strategy to the current work phase.

```
+-------------------------------------------------------------------------+
|                    INPUT ENHANCER ARCHITECTURE                           |
+-------------------------------------------------------------------------+
|                                                                         |
|  +-----------------+                                                    |
|  |   User Input    | --> Every message enters the pipeline              |
|  +--------+--------+                                                    |
|           |                                                             |
|           v                                                             |
|  +------------------------------------------------------------------+  |
|  |                    LAYER 1: Quick Filter                          |  |
|  |  Skip: <10 chars | !raw: prefix | slash commands | confirmations  |  |
|  +------------------------------------------------------------------+  |
|           |                                                             |
|           v                                                             |
|  +------------------------------------------------------------------+  |
|  |                    LAYER 2: Classifier                            |  |
|  |  11 types: debug | feature | explain | refactor | research        |  |
|  |            review | test | document | deploy | config | general   |  |
|  |  Output: type + confidence (0-100) + matched patterns             |  |
|  +------------------------------------------------------------------+  |
|           |                                                             |
|           v                                                             |
|  +------------------------------------------------------------------+  |
|  |                    LAYER 3: Plan Mode Detector                    |  |
|  |  Check ~/.claude/plans/ for recent files                          |  |
|  |  If active: shift to research-focused enhancement                 |  |
|  +------------------------------------------------------------------+  |
|           |                                                             |
|           v                                                             |
|  +------------------------------------------------------------------+  |
|  |                    LAYER 4: Complexity Detector                   |  |
|  |  Levels: simple | moderate | complex                              |  |
|  |  Complex triggers: security, multi-perspective, architecture      |  |
|  |  Output: suggestAgents boolean + agentTeamType                    |  |
|  +------------------------------------------------------------------+  |
|           |                                                             |
|           v                                                             |
|  +------------------------------------------------------------------+  |
|  |                    LAYER 5: Enhancement Generator                 |  |
|  |  Load template from EnhancementTemplates.yaml                     |  |
|  |  Apply plan_mode variant if in planning phase                     |  |
|  |  Generate chain-of-thought + constraints                          |  |
|  +------------------------------------------------------------------+  |
|           |                                                             |
|           v                                                             |
|  +------------------------------------------------------------------+  |
|  |                    LAYER 6: System Reminder Output                |  |
|  |  <system-reminder> with classification, recommendation, preview   |  |
|  |  Claude sees this and decides: apply directly or invoke skill     |  |
|  +------------------------------------------------------------------+  |
|           |                                                             |
|           +-------------+-------------------+                           |
|                         |                   |                           |
|                         v                   v                           |
|  +----------------------+--+  +---------------------------+            |
|  |  Simple: Apply Direct   |  |  Complex: Invoke Skill     |            |
|  |  Light CoT, proceed     |  |  PreviewEnhancement        |            |
|  +-------------------------+  |  ComposeAgentTeam          |            |
|                               |  AskClarification          |            |
|                               +---------------------------+            |
|                                                                         |
+-------------------------------------------------------------------------+
```

### How Data Flows Through the System

**Example: User asks "audit the API security"**

```
1. UserPromptSubmit fires with prompt: "audit the API security"
         |
         v
2. Quick filter: 22 chars, no bypass prefix -> CONTINUE
         |
         v
3. Classifier analyzes patterns:
   - "audit" -> review (weight: 15)
   - "security" -> security modifier (weight: 20)
   - Result: type=review, confidence=87%
         |
         v
4. Plan mode detector: no recent plan files -> NORMAL MODE
         |
         v
5. Complexity detector:
   - "security" keyword -> HIGH complexity
   - Multi-perspective beneficial -> suggestAgents=true
   - Agent team type: security-audit
         |
         v
6. Enhancement generator loads review template:
   <chain_of_thought>
   1. Map attack surface
   2. Check OWASP Top 10
   3. Review auth/authz
   4. Analyze data flow
   </chain_of_thought>
         |
         v
7. Hook outputs <system-reminder> with:
   - Type: review
   - Confidence: 87%
   - Complexity: HIGH
   - Suggestion: ComposeAgentTeam workflow
   - Enhancement preview
         |
         v
8. Claude sees reminder, recognizes complex security task
         |
         v
9. Claude invokes InputEnhancer skill -> ComposeAgentTeam workflow
         |
         v
10. Skill proposes 3-agent team:
    - Adversarial Tester
    - Security Architect
    - Compliance Auditor
         |
         v
11. User approves or modifies team
         |
         v
12. Agents spawn in parallel, each with enhanced prompt + CoT
```

### Why This Architecture Matters

**1. Hook Constraints Respected**
- Hooks CANNOT wait for user input (fire-and-forget)
- All interactive features go through the Skill
- Hook provides intelligence, Skill provides interaction

**2. Plan Mode Awareness**
- Detects when you're in planning phase
- Shifts from "how to implement" to "what to understand"
- Prevents premature implementation during research

**3. Progressive Enhancement**
- Simple inputs: light CoT, no interruption
- Complex inputs: full treatment via skill
- Ambiguous inputs: clarification questions

**4. Agent Integration**
- Complex tasks get multi-perspective analysis
- Pre-defined teams for common scenarios
- User can customize agents before launch

**5. Deterministic Classification**
- Pattern matching with explicit weights
- Confidence scores expose uncertainty
- Same input = same classification

### What Problems This Architecture Prevents

| Problem | How Input Enhancer Solves It |
|---------|------------------------------|
| Vague requests get vague responses | Classification adds structure |
| Complex tasks get shallow treatment | Multi-agent composition for depth |
| Plan mode gets implementation prompts | Plan mode detector shifts focus |
| User loses control over AI interpretation | Preview + edit before applying |
| Missing context causes wrong assumptions | Gap detection triggers questions |

---

## Why This Is Different

This sounds similar to Anthropic's Prompt Improver or DSPy which also enhance prompts. What makes this approach different?

Those systems require manual invocation or run as separate tools. They improve prompts but don't integrate with your AI workflow. The Kai Input Enhancer is wired directly into the event stream - every input is automatically classified and enhanced with zero friction. Plan mode awareness means enhancement adapts to your current work phase. Multi-agent composition means complex tasks get the team they need. And preview/edit via the skill means you stay in control.

- Automatic classification on every input with zero friction
- Plan mode shifts enhancement from implementation to research
- Multi-agent composition for complex multi-perspective tasks
- Preview and edit via skill keeps user in control

---

## Installation

### Prerequisites

- **Bun runtime**: `curl -fsSL https://bun.sh/install | bash`
- **Claude Code** (or compatible agent system with hook support)
- **Write access** to `$PAI_DIR/` (or your PAI directory)
- **kai-hook-system** installed (required dependency)
- **kai-core-install** installed (required for Skills structure)
- **kai-prompting-skill** installed (required for Standards.md principles)

---

### Pre-Installation: System Analysis

**IMPORTANT:** Before installing, analyze the current system state to detect conflicts and ensure dependencies are met.

#### Step 0.1: Detect Current Configuration

Run these commands to understand your current system:

```bash
# 1. Check if PAI_DIR is set
echo "PAI_DIR: ${PAI_DIR:-'NOT SET - will use ~/.config/pai'}"

# 2. Check for existing PAI directory
PAI_CHECK="${PAI_DIR:-$HOME/.config/pai}"
if [ -d "$PAI_CHECK" ]; then
  echo "PAI directory EXISTS at: $PAI_CHECK"
  echo "Contents:"
  ls -la "$PAI_CHECK" 2>/dev/null || echo "  (empty or inaccessible)"
else
  echo "PAI directory does not exist (clean install)"
fi

# 3. Check for existing input-enhancer files
echo ""
echo "Checking for files this pack will create..."
if [ -f "$PAI_CHECK/hooks/input-enhancer.ts" ]; then
  echo "WARNING: hooks/input-enhancer.ts already exists"
fi
if [ -d "$PAI_CHECK/hooks/lib/input-enhancer" ]; then
  echo "WARNING: hooks/lib/input-enhancer/ already exists"
fi
if [ -d "$PAI_CHECK/Skills/InputEnhancer" ]; then
  echo "WARNING: Skills/InputEnhancer/ already exists"
fi

# 4. Check Claude settings for existing hooks
CLAUDE_SETTINGS="$HOME/.claude/settings.json"
if [ -f "$CLAUDE_SETTINGS" ]; then
  echo "Claude settings.json EXISTS"
  if grep -q '"UserPromptSubmit"' "$CLAUDE_SETTINGS" 2>/dev/null; then
    echo "WARNING: Existing UserPromptSubmit hooks found - will need to MERGE"
  else
    echo "No UserPromptSubmit hooks configured"
  fi
else
  echo "No Claude settings.json (will be created)"
fi
```

#### Step 0.2: Verify Dependencies

```bash
PAI_CHECK="${PAI_DIR:-$HOME/.config/pai}"

# Check for required packs
echo "Checking required dependencies..."

# kai-hook-system
if [ -f "$PAI_CHECK/hooks/lib/observability.ts" ]; then
  echo "PASS: kai-hook-system is installed"
else
  echo "FAIL: kai-hook-system not installed (REQUIRED)"
  echo "  Install kai-hook-system first before proceeding"
fi

# kai-core-install (Skills directory)
if [ -d "$PAI_CHECK/Skills" ]; then
  echo "PASS: kai-core-install is installed (Skills/ exists)"
else
  echo "FAIL: kai-core-install not installed (REQUIRED)"
  echo "  Install kai-core-install first before proceeding"
fi

# kai-prompting-skill (for Standards.md)
if [ -f "$PAI_CHECK/Skills/Prompting/Context/Standards.md" ]; then
  echo "PASS: kai-prompting-skill is installed"
else
  echo "WARNING: kai-prompting-skill not installed"
  echo "  Some features may have reduced functionality"
fi

echo ""
echo "Checking optional dependencies..."

# kai-agents-skill (optional)
if [ -f "$PAI_CHECK/Skills/Agents/Tools/AgentFactory.ts" ]; then
  echo "PASS: kai-agents-skill is installed (agent composition available)"
else
  echo "INFO: kai-agents-skill not installed (agent composition disabled)"
fi

# kai-history-system (optional)
if [ -d "$PAI_CHECK/History" ]; then
  echo "PASS: kai-history-system is installed (logging available)"
else
  echo "INFO: kai-history-system not installed (logging disabled)"
fi
```

#### Step 0.3: Conflict Resolution Matrix

Based on the detection above, follow the appropriate path:

| Scenario | Existing State | Action |
|----------|---------------|--------|
| **Clean Install** | No conflicts | Proceed normally with Step 1 |
| **Hook Exists** | input-enhancer.ts present | Backup old file, then replace |
| **Skill Exists** | InputEnhancer/ present | Backup directory, then replace |
| **UserPromptSubmit Hooks** | Other hooks on event | **MERGE** - add new hook to existing array |
| **Missing Dependencies** | Required packs missing | Install dependencies first |

#### Step 0.4: Backup Existing Configuration (If Needed)

If conflicts were detected, create a backup before proceeding:

```bash
# Create timestamped backup
BACKUP_DIR="$HOME/.pai-backup/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"
PAI_CHECK="${PAI_DIR:-$HOME/.config/pai}"

# Backup files this pack will modify
if [ -f "$PAI_CHECK/hooks/input-enhancer.ts" ]; then
  cp "$PAI_CHECK/hooks/input-enhancer.ts" "$BACKUP_DIR/"
  echo "Backed up input-enhancer.ts"
fi

if [ -d "$PAI_CHECK/hooks/lib/input-enhancer" ]; then
  cp -r "$PAI_CHECK/hooks/lib/input-enhancer" "$BACKUP_DIR/lib-input-enhancer"
  echo "Backed up hooks/lib/input-enhancer/"
fi

if [ -d "$PAI_CHECK/Skills/InputEnhancer" ]; then
  cp -r "$PAI_CHECK/Skills/InputEnhancer" "$BACKUP_DIR/"
  echo "Backed up Skills/InputEnhancer/"
fi

echo "Backup location: $BACKUP_DIR"
```

**After completing system analysis, proceed to Step 1.**

---

### Step 1: Create Directory Structure

```bash
PAI_DIR="${PAI_DIR:-$HOME/.config/pai}"

# Create all required directories
mkdir -p "$PAI_DIR/hooks/lib/input-enhancer"
mkdir -p "$PAI_DIR/Skills/InputEnhancer/Workflows"
mkdir -p "$PAI_DIR/Skills/InputEnhancer/Data"
mkdir -p "$PAI_DIR/Skills/InputEnhancer/Tools"
mkdir -p "$PAI_DIR/History/enhancements"

# Verify structure
echo "Created directories:"
ls -la "$PAI_DIR/hooks/lib/"
ls -la "$PAI_DIR/Skills/InputEnhancer/"
```

Expected output: All directories created with no errors.

---

### Step 2: Create Library Files

These shared libraries are used by the main hook.

#### 2.1: Create types.ts

```typescript
// $PAI_DIR/hooks/lib/input-enhancer/types.ts
// Type definitions for input enhancement

export type RequestType =
  | 'debug'
  | 'feature'
  | 'explain'
  | 'refactor'
  | 'research'
  | 'review'
  | 'test'
  | 'document'
  | 'deploy'
  | 'config'
  | 'general';

export interface Classification {
  type: RequestType;
  confidence: number; // 0-100
  matchedPatterns: string[];
}

export interface Gap {
  type: 'scope' | 'target' | 'goal' | 'interpretation';
  description: string;
  severity: 'low' | 'medium' | 'high';
}

export interface ComplexityAnalysis {
  level: 'simple' | 'moderate' | 'complex';
  suggestAgents: boolean;
  agentTeamType?: 'security' | 'review' | 'research' | 'analysis';
  gaps: Gap[];
  reasons: string[];
}

export interface PlanModeStatus {
  active: boolean;
  planFile?: string;
  phase?: 'exploring' | 'designing' | 'reviewing';
}

export interface EnhancementResult {
  classification: Classification;
  complexity: ComplexityAnalysis;
  planMode: PlanModeStatus;
  enhancement: {
    chainOfThought?: string[];
    constraints?: {
      success: string;
      failure: string;
    };
    structure?: string[];
  };
  recommendation: {
    action: 'apply_direct' | 'invoke_skill' | 'skip';
    workflow?: 'PreviewEnhancement' | 'ComposeAgentTeam' | 'AskClarification';
    reason: string;
  };
}

export interface HookInput {
  session_id: string;
  prompt: string;
  transcript_path?: string;
}
```

#### 2.2: Create quick-filter.ts

```typescript
// $PAI_DIR/hooks/lib/input-enhancer/quick-filter.ts
// Skip logic for short/bypass/command inputs

export interface FilterResult {
  shouldSkip: boolean;
  reason?: string;
}

// Bypass prefixes that skip enhancement
const BYPASS_PREFIXES = ['!raw:', '*', '/'];

// Common confirmations that don't need enhancement
const CONFIRMATIONS = [
  'yes', 'no', 'ok', 'okay', 'sure', 'yep', 'nope',
  'y', 'n', 'continue', 'proceed', 'go ahead', 'do it',
  'cancel', 'stop', 'quit', 'exit', 'done', 'thanks',
  'thank you', 'thx', 'ty', 'please', 'pls'
];

// Minimum length to consider for enhancement
const MIN_LENGTH = 10;

/**
 * Determine if input should skip enhancement
 */
export function shouldSkipEnhancement(prompt: string): FilterResult {
  // Check environment variable override
  if (process.env.PAI_INPUT_ENHANCE === 'false') {
    return { shouldSkip: true, reason: 'PAI_INPUT_ENHANCE=false' };
  }

  const trimmed = prompt.trim();
  const lower = trimmed.toLowerCase();

  // Skip very short inputs
  if (trimmed.length < MIN_LENGTH) {
    return { shouldSkip: true, reason: `Input too short (${trimmed.length} < ${MIN_LENGTH})` };
  }

  // Skip bypass prefixes
  for (const prefix of BYPASS_PREFIXES) {
    if (trimmed.startsWith(prefix)) {
      return { shouldSkip: true, reason: `Bypass prefix: ${prefix}` };
    }
  }

  // Skip common confirmations
  if (CONFIRMATIONS.includes(lower)) {
    return { shouldSkip: true, reason: 'Common confirmation' };
  }

  // Skip if already has markdown structure (## headers)
  if (trimmed.includes('\n##') || trimmed.startsWith('##')) {
    return { shouldSkip: true, reason: 'Already has markdown structure' };
  }

  // Skip if contains XML-like tags (already structured)
  if (/<[a-z_]+>/.test(trimmed) && /<\/[a-z_]+>/.test(trimmed)) {
    return { shouldSkip: true, reason: 'Already has XML structure' };
  }

  return { shouldSkip: false };
}
```

#### 2.3: Create classifier.ts

```typescript
// $PAI_DIR/hooks/lib/input-enhancer/classifier.ts
// Classification engine for request types

import type { RequestType, Classification } from './types';

interface PatternMatch {
  type: RequestType;
  patterns: RegExp[];
  weight: number;
}

// Pattern definitions for each request type
const PATTERN_MATCHERS: PatternMatch[] = [
  {
    type: 'debug',
    patterns: [
      /\b(fix|bug|error|issue|problem|broken|crash|fail|not working)\b/i,
      /\b(debug|trace|diagnose|troubleshoot)\b/i,
      /\b(TypeError|ReferenceError|SyntaxError|Exception)\b/,
      /\bline\s+\d+\b/i,
      /\b(undefined|null|NaN)\s+(is not|error)\b/i
    ],
    weight: 15
  },
  {
    type: 'feature',
    patterns: [
      /\b(add|create|implement|build|make|new)\b/i,
      /\b(feature|functionality|capability)\b/i,
      /\b(want|need|should have|would like)\b/i,
      /\b(button|form|page|component|endpoint|api)\b/i
    ],
    weight: 12
  },
  {
    type: 'explain',
    patterns: [
      /\b(explain|describe|what is|what does|how does|why does)\b/i,
      /\b(understand|clarify|elaborate|meaning)\b/i,
      /\b(walk me through|tell me about)\b/i,
      /\?$/
    ],
    weight: 10
  },
  {
    type: 'refactor',
    patterns: [
      /\b(refactor|restructure|reorganize|clean up|improve)\b/i,
      /\b(simplify|optimize|modernize|update)\b/i,
      /\b(code quality|readability|maintainability)\b/i,
      /\b(DRY|SOLID|pattern)\b/i
    ],
    weight: 12
  },
  {
    type: 'research',
    patterns: [
      /\b(research|investigate|explore|find out|look into)\b/i,
      /\b(compare|evaluate|assess|analyze)\b/i,
      /\b(best practice|recommendation|option|alternative)\b/i,
      /\b(library|framework|tool|approach)\b/i
    ],
    weight: 10
  },
  {
    type: 'review',
    patterns: [
      /\b(review|audit|check|verify|validate)\b/i,
      /\b(security|vulnerability|risk|threat)\b/i,
      /\b(code review|pull request|PR|merge)\b/i,
      /\b(compliance|standard|best practice)\b/i
    ],
    weight: 15
  },
  {
    type: 'test',
    patterns: [
      /\b(test|spec|coverage|assertion)\b/i,
      /\b(unit test|integration test|e2e|end-to-end)\b/i,
      /\b(mock|stub|fixture|setup)\b/i,
      /\b(jest|vitest|mocha|pytest)\b/i
    ],
    weight: 12
  },
  {
    type: 'document',
    patterns: [
      /\b(document|documentation|readme|docs)\b/i,
      /\b(comment|jsdoc|docstring|annotation)\b/i,
      /\b(explain|describe)\s+(the|this|how)\b/i,
      /\b(api doc|spec|guide)\b/i
    ],
    weight: 10
  },
  {
    type: 'deploy',
    patterns: [
      /\b(deploy|release|publish|ship)\b/i,
      /\b(CI\/CD|pipeline|workflow|action)\b/i,
      /\b(docker|kubernetes|k8s|container)\b/i,
      /\b(production|staging|environment)\b/i
    ],
    weight: 12
  },
  {
    type: 'config',
    patterns: [
      /\b(config|configure|setup|setting)\b/i,
      /\b(environment|env|variable)\b/i,
      /\b(\.env|json|yaml|toml)\b/i,
      /\b(install|dependency|package)\b/i
    ],
    weight: 10
  }
];

// Modifier patterns that boost confidence
const MODIFIERS = {
  security: { pattern: /\b(security|secure|vulnerability|attack|exploit|auth)\b/i, boost: 20 },
  urgent: { pattern: /\b(urgent|critical|asap|immediately|now)\b/i, boost: 5 },
  specific: { pattern: /\b(specifically|exactly|precisely|this)\b/i, boost: 8 },
  file: { pattern: /\.(ts|js|py|go|rs|java|tsx|jsx|vue|svelte)\b/i, boost: 10 }
};

/**
 * Classify a prompt into one of the 11 request types
 */
export function classifyPrompt(prompt: string): Classification {
  const scores: Record<RequestType, { score: number; patterns: string[] }> = {
    debug: { score: 0, patterns: [] },
    feature: { score: 0, patterns: [] },
    explain: { score: 0, patterns: [] },
    refactor: { score: 0, patterns: [] },
    research: { score: 0, patterns: [] },
    review: { score: 0, patterns: [] },
    test: { score: 0, patterns: [] },
    document: { score: 0, patterns: [] },
    deploy: { score: 0, patterns: [] },
    config: { score: 0, patterns: [] },
    general: { score: 0, patterns: [] }
  };

  // Score each type based on pattern matches
  for (const matcher of PATTERN_MATCHERS) {
    for (const pattern of matcher.patterns) {
      if (pattern.test(prompt)) {
        scores[matcher.type].score += matcher.weight;
        scores[matcher.type].patterns.push(pattern.source);
      }
    }
  }

  // Apply modifier boosts
  for (const [name, mod] of Object.entries(MODIFIERS)) {
    if (mod.pattern.test(prompt)) {
      // Boost the highest scoring type
      const topType = Object.entries(scores)
        .filter(([type]) => type !== 'general')
        .sort(([, a], [, b]) => b.score - a.score)[0];

      if (topType && topType[1].score > 0) {
        topType[1].score += mod.boost;
        topType[1].patterns.push(`modifier:${name}`);
      }
    }
  }

  // Find the highest scoring type
  const sorted = Object.entries(scores)
    .filter(([type]) => type !== 'general')
    .sort(([, a], [, b]) => b.score - a.score);

  const [topType, topScore] = sorted[0] || ['general', { score: 0, patterns: [] }];

  // Calculate confidence (0-100)
  // Base confidence from score, capped at 95
  const baseConfidence = Math.min(topScore.score * 2.5, 95);

  // If no patterns matched, return general with low confidence
  if (topScore.score === 0) {
    return {
      type: 'general',
      confidence: 25,
      matchedPatterns: []
    };
  }

  return {
    type: topType as RequestType,
    confidence: Math.round(baseConfidence),
    matchedPatterns: topScore.patterns
  };
}
```

#### 2.4: Create complexity-detector.ts

```typescript
// $PAI_DIR/hooks/lib/input-enhancer/complexity-detector.ts
// Detect task complexity and when agents would help

import type { Classification, ComplexityAnalysis, Gap } from './types';

// Keywords that suggest high complexity
const COMPLEXITY_KEYWORDS = {
  high: [
    /\b(security|audit|vulnerability|penetration|attack)\b/i,
    /\b(architecture|design|system|infrastructure)\b/i,
    /\b(comprehensive|thorough|complete|full)\b/i,
    /\b(multiple|several|all|every|entire)\b/i,
    /\b(trade-?off|decision|choice|option)\b/i,
    /\b(performance|optimization|scalability)\b/i
  ],
  moderate: [
    /\b(review|check|verify|validate)\b/i,
    /\b(compare|evaluate|analyze)\b/i,
    /\b(refactor|restructure|reorganize)\b/i,
    /\b(integrate|connect|combine)\b/i
  ]
};

// Agent team recommendations by scenario
const AGENT_SCENARIOS = {
  'security': {
    keywords: [/\b(security|audit|vulnerability|penetration|auth)\b/i],
    teamType: 'security' as const,
    reason: 'Security tasks benefit from adversarial, defensive, and compliance perspectives'
  },
  'review': {
    keywords: [/\b(review|code review|PR|pull request|audit)\b/i],
    teamType: 'review' as const,
    reason: 'Code reviews benefit from bug-hunting, architecture, and performance perspectives'
  },
  'research': {
    keywords: [/\b(research|investigate|explore|compare|evaluate)\b/i],
    teamType: 'research' as const,
    reason: 'Research tasks benefit from exploration, critique, and synthesis perspectives'
  },
  'analysis': {
    keywords: [/\b(analyze|assessment|decision|trade-?off|architecture)\b/i],
    teamType: 'analysis' as const,
    reason: 'Analysis tasks benefit from multiple expert perspectives'
  }
};

// Gap detection patterns
const GAP_PATTERNS = {
  scope: {
    patterns: [
      /^(fix|update|change|modify)\s+(it|this|that)$/i,
      /^(do|make|help)\s+(it|this|that)/i
    ],
    description: 'Missing scope - unclear what part of the system is involved',
    severity: 'high' as const
  },
  target: {
    patterns: [
      /^(fix|update)\s+the\s+(bug|error|issue)$/i,
      /\b(the file|the code|the function)\b(?!\s+\w+\.)/i
    ],
    description: 'Missing target - no specific file or component mentioned',
    severity: 'medium' as const
  },
  goal: {
    patterns: [
      /^(improve|make better|optimize)$/i,
      /^(help|assist|support)$/i
    ],
    description: 'Missing goal - unclear what success looks like',
    severity: 'high' as const
  },
  interpretation: {
    patterns: [
      /\b(or|maybe|perhaps|possibly)\b/i,
      /\b(either|any|whatever)\b/i
    ],
    description: 'Multiple interpretations possible',
    severity: 'low' as const
  }
};

/**
 * Analyze task complexity and determine if agents would help
 */
export function analyzeComplexity(
  prompt: string,
  classification: Classification
): ComplexityAnalysis {
  const reasons: string[] = [];
  const gaps: Gap[] = [];
  let complexityScore = 0;

  // Check for high complexity keywords
  for (const pattern of COMPLEXITY_KEYWORDS.high) {
    if (pattern.test(prompt)) {
      complexityScore += 2;
      reasons.push(`High complexity keyword: ${pattern.source}`);
    }
  }

  // Check for moderate complexity keywords
  for (const pattern of COMPLEXITY_KEYWORDS.moderate) {
    if (pattern.test(prompt)) {
      complexityScore += 1;
      reasons.push(`Moderate complexity keyword: ${pattern.source}`);
    }
  }

  // Check classification type
  if (['review', 'research'].includes(classification.type)) {
    complexityScore += 1;
    reasons.push(`Classification type suggests complexity: ${classification.type}`);
  }

  // Detect gaps
  for (const [gapType, config] of Object.entries(GAP_PATTERNS)) {
    for (const pattern of config.patterns) {
      if (pattern.test(prompt)) {
        gaps.push({
          type: gapType as Gap['type'],
          description: config.description,
          severity: config.severity
        });
        if (config.severity === 'high') {
          complexityScore += 1;
        }
        break; // Only count each gap type once
      }
    }
  }

  // Low confidence classification adds complexity
  if (classification.confidence < 50) {
    complexityScore += 1;
    reasons.push('Low classification confidence suggests ambiguity');
  }

  // Determine complexity level
  let level: ComplexityAnalysis['level'];
  if (complexityScore >= 4) {
    level = 'complex';
  } else if (complexityScore >= 2) {
    level = 'moderate';
  } else {
    level = 'simple';
  }

  // Determine if agents would help
  let suggestAgents = false;
  let agentTeamType: ComplexityAnalysis['agentTeamType'];

  for (const [scenario, config] of Object.entries(AGENT_SCENARIOS)) {
    for (const pattern of config.keywords) {
      if (pattern.test(prompt)) {
        suggestAgents = true;
        agentTeamType = config.teamType;
        reasons.push(config.reason);
        break;
      }
    }
    if (suggestAgents) break;
  }

  // Complex tasks without specific agent match still suggest agents
  if (!suggestAgents && level === 'complex') {
    suggestAgents = true;
    agentTeamType = 'analysis';
    reasons.push('Complex task would benefit from multiple perspectives');
  }

  return {
    level,
    suggestAgents,
    agentTeamType,
    gaps,
    reasons
  };
}
```

#### 2.5: Create plan-mode-detector.ts

```typescript
// $PAI_DIR/hooks/lib/input-enhancer/plan-mode-detector.ts
// Detect if plan mode is active and adjust enhancement strategy

import { existsSync, readdirSync, statSync } from 'fs';
import { join } from 'path';
import type { PlanModeStatus } from './types';

const HOME = process.env.HOME || '/root';
const PLANS_DIR = join(HOME, '.claude', 'plans');

// How recent a plan file must be to consider plan mode active (minutes)
const RECENT_THRESHOLD_MINUTES = 30;

/**
 * Check if a file was modified within the threshold
 */
function isRecentlyModified(filePath: string, thresholdMinutes: number): boolean {
  try {
    const stats = statSync(filePath);
    const now = Date.now();
    const modified = stats.mtime.getTime();
    const diffMinutes = (now - modified) / (1000 * 60);
    return diffMinutes <= thresholdMinutes;
  } catch {
    return false;
  }
}

/**
 * Determine the planning phase based on file content
 */
function detectPhase(filePath: string): PlanModeStatus['phase'] {
  try {
    const content = Bun.file(filePath).text();
    // Simple heuristics based on content
    if (content.includes('## Implementation') || content.includes('## Code')) {
      return 'reviewing';
    }
    if (content.includes('## Design') || content.includes('## Architecture')) {
      return 'designing';
    }
    return 'exploring';
  } catch {
    return 'exploring';
  }
}

/**
 * Detect if plan mode is currently active
 */
export async function detectPlanMode(): Promise<PlanModeStatus> {
  // Check 1: Environment variable override
  if (process.env.CLAUDE_PLAN_MODE === 'true') {
    return { active: true, phase: 'designing' };
  }

  if (process.env.CLAUDE_PLAN_MODE === 'false') {
    return { active: false };
  }

  // Check 2: Recent plan file exists
  if (existsSync(PLANS_DIR)) {
    try {
      const files = readdirSync(PLANS_DIR);
      const planFiles = files.filter(f => f.endsWith('.md'));

      for (const file of planFiles) {
        const filePath = join(PLANS_DIR, file);
        if (isRecentlyModified(filePath, RECENT_THRESHOLD_MINUTES)) {
          return {
            active: true,
            planFile: file,
            phase: detectPhase(filePath)
          };
        }
      }
    } catch {
      // Directory exists but can't be read - not in plan mode
    }
  }

  return { active: false };
}

/**
 * Get plan mode adjustment recommendations
 */
export function getPlanModeAdjustments(planMode: PlanModeStatus): {
  cotFocus: string;
  agentFocus: string;
  questionFocus: string;
  constraintFocus: string;
} {
  if (!planMode.active) {
    return {
      cotFocus: 'implementation',
      agentFocus: 'Team to do the work',
      questionFocus: 'What to build?',
      constraintFocus: 'Implementation success'
    };
  }

  return {
    cotFocus: 'research and exploration',
    agentFocus: 'Team to explore options',
    questionFocus: 'What to understand first?',
    constraintFocus: 'Understanding completeness'
  };
}
```

#### 2.6: Create enhancement-generator.ts

```typescript
// $PAI_DIR/hooks/lib/input-enhancer/enhancement-generator.ts
// Generate chain-of-thought enhancements based on classification

import { parse as parseYaml } from 'yaml';
import type { Classification, ComplexityAnalysis, PlanModeStatus, EnhancementResult } from './types';

const PAI_DIR = process.env.PAI_DIR || `${process.env.HOME}/.config/pai`;

interface EnhancementTemplate {
  cot: boolean;
  chain_of_thought?: string[];
  constraints?: {
    success: string;
    failure: string;
  };
  structure?: string[];
  plan_mode?: {
    chain_of_thought?: string[];
    constraints?: {
      success: string;
      failure: string;
    };
  };
}

interface TemplatesConfig {
  templates: Record<string, EnhancementTemplate>;
}

// Default templates if file not found
const DEFAULT_TEMPLATES: TemplatesConfig = {
  templates: {
    debug: {
      cot: true,
      chain_of_thought: [
        'Identify symptoms and error messages',
        'Determine expected vs actual behavior',
        'Trace code flow to find failure point',
        'Identify root cause (not just symptoms)',
        'Propose fix with verification steps'
      ],
      constraints: {
        success: 'Root cause identified and fixed',
        failure: 'Symptom masked without understanding'
      },
      plan_mode: {
        chain_of_thought: [
          'What are all the reported symptoms?',
          'What code paths could cause this behavior?',
          'What similar bugs exist in the codebase history?',
          'What testing gaps allowed this to occur?',
          'What are the possible fix approaches and trade-offs?'
        ],
        constraints: {
          success: 'Bug fully understood with fix options documented',
          failure: 'Jumping to fix without understanding'
        }
      }
    },
    feature: {
      cot: true,
      chain_of_thought: [
        'Clarify requirements and acceptance criteria',
        'Identify where in codebase this belongs',
        'Consider edge cases and error handling',
        'Implement with minimal complexity',
        'Verify feature works as intended'
      ],
      constraints: {
        success: 'Feature implemented and tested',
        failure: 'Over-engineered or missing requirements'
      },
      plan_mode: {
        chain_of_thought: [
          'What exactly should this feature do?',
          'How do similar features work in this codebase?',
          'What are the architectural options?',
          'What dependencies and integrations are needed?',
          'What are the risks and unknowns?'
        ],
        constraints: {
          success: 'Feature fully scoped with clear implementation plan',
          failure: 'Starting implementation without understanding scope'
        }
      }
    },
    explain: {
      cot: false,
      structure: [
        'High-level overview',
        'Key components and relationships',
        'Concrete examples',
        'Summary of key points'
      ]
    },
    refactor: {
      cot: true,
      chain_of_thought: [
        'Identify code smells and issues',
        'Determine target structure',
        'Plan incremental changes',
        'Implement with tests passing',
        'Verify behavior unchanged'
      ],
      constraints: {
        success: 'Code improved without breaking functionality',
        failure: 'Introduced bugs or broke existing features'
      },
      plan_mode: {
        chain_of_thought: [
          'What specific issues need addressing?',
          'What patterns exist in similar code?',
          'What are the refactoring options?',
          'What risks does each approach carry?',
          'How to verify nothing breaks?'
        ],
        constraints: {
          success: 'Refactoring plan with clear steps and verification',
          failure: 'Starting changes without understanding impact'
        }
      }
    },
    research: {
      cot: true,
      chain_of_thought: [
        'Define research questions clearly',
        'Identify reliable sources',
        'Gather diverse perspectives',
        'Synthesize findings',
        'Provide actionable recommendations'
      ],
      constraints: {
        success: 'Comprehensive understanding with clear next steps',
        failure: 'Surface-level findings without depth'
      }
    },
    review: {
      cot: true,
      chain_of_thought: [
        'Understand the scope and purpose',
        'Check for bugs and edge cases',
        'Evaluate design and patterns',
        'Consider security implications',
        'Provide constructive feedback'
      ],
      constraints: {
        success: 'Thorough review with actionable feedback',
        failure: 'Superficial check missing important issues'
      },
      plan_mode: {
        chain_of_thought: [
          'What is the scope of this review?',
          'What standards and patterns apply?',
          'What are the key risk areas?',
          'What review checklist should be used?',
          'How to prioritize findings?'
        ],
        constraints: {
          success: 'Review strategy defined with clear criteria',
          failure: 'Starting review without clear focus'
        }
      }
    },
    test: {
      cot: true,
      chain_of_thought: [
        'Identify what needs testing',
        'Define test cases and edge cases',
        'Set up test fixtures',
        'Implement tests',
        'Verify coverage'
      ],
      constraints: {
        success: 'Comprehensive test coverage',
        failure: 'Tests that pass but miss important cases'
      }
    },
    document: {
      cot: false,
      structure: [
        'Purpose and overview',
        'Installation/setup',
        'Usage examples',
        'API reference (if applicable)',
        'Troubleshooting'
      ]
    },
    deploy: {
      cot: true,
      chain_of_thought: [
        'Verify code is ready for deployment',
        'Check environment configuration',
        'Plan rollback strategy',
        'Execute deployment',
        'Verify deployment success'
      ],
      constraints: {
        success: 'Successful deployment with verification',
        failure: 'Deployment issues or missing rollback plan'
      }
    },
    config: {
      cot: true,
      chain_of_thought: [
        'Understand configuration requirements',
        'Identify affected components',
        'Make changes incrementally',
        'Test configuration',
        'Document changes'
      ],
      constraints: {
        success: 'Configuration working and documented',
        failure: 'Misconfiguration causing runtime issues'
      }
    },
    general: {
      cot: false,
      structure: [
        'Understand the request',
        'Break into steps if needed',
        'Execute methodically',
        'Verify completion'
      ]
    }
  }
};

/**
 * Load enhancement templates from YAML file
 */
async function loadTemplates(): Promise<TemplatesConfig> {
  try {
    const templatePath = `${PAI_DIR}/Skills/InputEnhancer/Data/EnhancementTemplates.yaml`;
    const content = await Bun.file(templatePath).text();
    return parseYaml(content) as TemplatesConfig;
  } catch {
    return DEFAULT_TEMPLATES;
  }
}

/**
 * Generate enhancement based on classification and context
 */
export async function generateEnhancement(
  prompt: string,
  classification: Classification,
  complexity: ComplexityAnalysis,
  planMode: PlanModeStatus
): Promise<EnhancementResult> {
  const templates = await loadTemplates();
  const template = templates.templates[classification.type] || templates.templates.general;

  // Determine which template variant to use
  let chainOfThought = template.chain_of_thought;
  let constraints = template.constraints;

  // Apply plan mode variant if active
  if (planMode.active && template.plan_mode) {
    if (template.plan_mode.chain_of_thought) {
      chainOfThought = template.plan_mode.chain_of_thought;
    }
    if (template.plan_mode.constraints) {
      constraints = template.plan_mode.constraints;
    }
  }

  // Build enhancement result
  const enhancement: EnhancementResult['enhancement'] = {};

  if (template.cot && chainOfThought) {
    enhancement.chainOfThought = chainOfThought;
  }

  if (constraints) {
    enhancement.constraints = constraints;
  }

  if (template.structure) {
    enhancement.structure = template.structure;
  }

  // Determine recommendation
  let action: EnhancementResult['recommendation']['action'];
  let workflow: EnhancementResult['recommendation']['workflow'];
  let reason: string;

  if (complexity.gaps.some(g => g.severity === 'high')) {
    action = 'invoke_skill';
    workflow = 'AskClarification';
    reason = 'High severity gaps detected - need clarification';
  } else if (complexity.suggestAgents) {
    action = 'invoke_skill';
    workflow = 'ComposeAgentTeam';
    reason = `Complex task would benefit from ${complexity.agentTeamType} agent team`;
  } else if (complexity.level === 'complex' || classification.confidence < 60) {
    action = 'invoke_skill';
    workflow = 'PreviewEnhancement';
    reason = 'Complex or ambiguous - preview enhancement first';
  } else if (complexity.level === 'simple' && classification.confidence >= 80) {
    action = 'apply_direct';
    reason = 'Simple task with high confidence - apply directly';
  } else {
    action = 'apply_direct';
    reason = 'Moderate complexity - apply light enhancement';
  }

  return {
    classification,
    complexity,
    planMode,
    enhancement,
    recommendation: { action, workflow, reason }
  };
}

/**
 * Format enhancement as XML for system reminder
 */
export function formatEnhancementXml(result: EnhancementResult): string {
  const lines: string[] = [];

  if (result.enhancement.chainOfThought) {
    lines.push('<chain_of_thought>');
    result.enhancement.chainOfThought.forEach((step, i) => {
      lines.push(`${i + 1}. ${step}`);
    });
    lines.push('</chain_of_thought>');
  }

  if (result.enhancement.constraints) {
    lines.push('');
    lines.push('<constraints>');
    lines.push(`Success: ${result.enhancement.constraints.success}`);
    lines.push(`Failure: ${result.enhancement.constraints.failure}`);
    lines.push('</constraints>');
  }

  if (result.enhancement.structure) {
    lines.push('<structure>');
    result.enhancement.structure.forEach((section, i) => {
      lines.push(`${i + 1}. ${section}`);
    });
    lines.push('</structure>');
  }

  return lines.join('\n');
}
```

---

### Step 3: Create Main Hook

#### 3.1: Create input-enhancer.ts

```typescript
// $PAI_DIR/hooks/input-enhancer.ts
// Main hook for input enhancement - fires on UserPromptSubmit

import { shouldSkipEnhancement } from './lib/input-enhancer/quick-filter';
import { classifyPrompt } from './lib/input-enhancer/classifier';
import { analyzeComplexity } from './lib/input-enhancer/complexity-detector';
import { detectPlanMode, getPlanModeAdjustments } from './lib/input-enhancer/plan-mode-detector';
import { generateEnhancement, formatEnhancementXml } from './lib/input-enhancer/enhancement-generator';
import type { HookInput } from './lib/input-enhancer/types';

/**
 * Main hook entry point
 */
async function main() {
  try {
    // Read input from stdin
    const input = await Bun.stdin.text();
    if (!input.trim()) {
      process.exit(0);
    }

    const hookInput: HookInput = JSON.parse(input);
    const prompt = hookInput.prompt;

    if (!prompt) {
      process.exit(0);
    }

    // Step 1: Quick filter
    const filterResult = shouldSkipEnhancement(prompt);
    if (filterResult.shouldSkip) {
      // Silent exit - no output needed
      process.exit(0);
    }

    // Step 2: Classify the prompt
    const classification = classifyPrompt(prompt);

    // Step 3: Detect plan mode
    const planMode = await detectPlanMode();
    const planAdjustments = getPlanModeAdjustments(planMode);

    // Step 4: Analyze complexity
    const complexity = analyzeComplexity(prompt, classification);

    // Step 5: Generate enhancement
    const result = await generateEnhancement(prompt, classification, complexity, planMode);

    // Step 6: Format output as system-reminder
    const output = formatSystemReminder(result, planAdjustments);

    // Output to stdout for Claude to see
    console.log(output);

    process.exit(0);
  } catch (error) {
    // Fail silently - hooks should never crash Claude Code
    // Optionally log to stderr for debugging
    if (process.env.PAI_DEBUG === 'true') {
      console.error('[InputEnhancer] Error:', error);
    }
    process.exit(0);
  }
}

/**
 * Format the full system reminder output
 */
function formatSystemReminder(
  result: Awaited<ReturnType<typeof generateEnhancement>>,
  planAdjustments: ReturnType<typeof getPlanModeAdjustments>
): string {
  const { classification, complexity, planMode, recommendation } = result;

  const lines: string[] = [];
  lines.push('<system-reminder>');
  lines.push('[InputEnhancer] Analysis Complete');
  lines.push('');

  // Plan mode indicator
  if (planMode.active) {
    lines.push(`Mode: PLAN MODE (${planMode.phase || 'active'})`);
    lines.push(`Focus: ${planAdjustments.cotFocus}`);
    lines.push('');
  }

  // Classification
  lines.push(`Type: ${classification.type}`);
  lines.push(`Confidence: ${classification.confidence}%`);
  lines.push(`Complexity: ${complexity.level.toUpperCase()}`);

  // Gaps detected
  if (complexity.gaps.length > 0) {
    lines.push('');
    lines.push('Gaps Detected:');
    for (const gap of complexity.gaps) {
      lines.push(`- [${gap.severity.toUpperCase()}] ${gap.type}: ${gap.description}`);
    }
  }

  // Recommendation
  lines.push('');
  lines.push('Recommendation:');
  lines.push(`- Action: ${recommendation.action.replace('_', ' ')}`);
  if (recommendation.workflow) {
    lines.push(`- Suggested workflow: ${recommendation.workflow}`);
  }
  lines.push(`- Reason: ${recommendation.reason}`);

  // Agent suggestion
  if (complexity.suggestAgents && complexity.agentTeamType) {
    lines.push('');
    lines.push(`Agent Team Suggested: ${complexity.agentTeamType}`);
    lines.push(`${planAdjustments.agentFocus}`);
  }

  // Enhancement preview
  const enhancementXml = formatEnhancementXml(result);
  if (enhancementXml) {
    lines.push('');
    lines.push('Enhancement Preview:');
    lines.push('<enhancement>');
    lines.push(enhancementXml);
    lines.push('</enhancement>');
  }

  // Action guidance
  lines.push('');
  if (recommendation.action === 'apply_direct') {
    lines.push('Action: Apply enhancement directly and proceed with the task.');
  } else if (recommendation.workflow === 'AskClarification') {
    lines.push('Action: Ask clarifying questions before proceeding.');
  } else if (recommendation.workflow === 'ComposeAgentTeam') {
    lines.push('Action: Consider proposing an agent team for multi-perspective analysis.');
  } else {
    lines.push('Action: Consider showing enhancement preview to user before applying.');
  }

  lines.push('</system-reminder>');

  return lines.join('\n');
}

// Run main
main();
```

---

### Step 4: Create Skill Files

#### 4.1: Create SKILL.md

```markdown
<!-- $PAI_DIR/Skills/InputEnhancer/SKILL.md -->
---
name: InputEnhancer
description: Input enhancement with preview, editing, and agent composition.
  USE WHEN the input-enhancer hook recommends skill invocation,
  OR when user explicitly asks to "enhance this", "preview enhancement",
  "compose agents", or "clarify my request".
---

# InputEnhancer Skill

The InputEnhancer skill provides interactive features that the automatic hook cannot provide (because hooks can't wait for user input).

## When This Skill Is Invoked

1. **Hook recommends skill invocation** - For complex or ambiguous tasks
2. **User requests**: "enhance this", "preview enhancement", "show enhancement"
3. **User requests**: "compose agents", "agent team", "multi-agent analysis"
4. **User requests**: "clarify", "ask questions about my request"

## Workflow Routing

| Trigger | Workflow | Description |
|---------|----------|-------------|
| Complex task, user wants preview | PreviewEnhancement | Show enhancement, allow editing before applying |
| Task benefits from agents | ComposeAgentTeam | Propose specialized agent team |
| Ambiguous input, gaps detected | AskClarification | Ask targeted questions |

## How to Use

Based on the hook's `<system-reminder>` output:

1. **If `recommendation.action = apply_direct`**: Apply enhancement and proceed
2. **If `recommendation.workflow = PreviewEnhancement`**: Follow Workflows/PreviewEnhancement.md
3. **If `recommendation.workflow = ComposeAgentTeam`**: Follow Workflows/ComposeAgentTeam.md
4. **If `recommendation.workflow = AskClarification`**: Follow Workflows/AskClarification.md

## Available Data

- `Data/EnhancementTemplates.yaml` - CoT templates by request type
- `Data/AgentProfiles.yaml` - Pre-defined agent teams
- `Data/QuestionPatterns.yaml` - Questions for gap types

## Available Tools

- `Tools/GenerateEnhancement.ts` - CLI for generating enhancements

## Integration Points

- **kai-agents-skill**: For agent composition (optional)
- **kai-history-system**: For logging (optional)
- **kai-prompting-skill**: For Standards.md principles
```

#### 4.2: Create PreviewEnhancement.md

```markdown
<!-- $PAI_DIR/Skills/InputEnhancer/Workflows/PreviewEnhancement.md -->
# PreviewEnhancement Workflow

Show the proposed enhancement to the user and allow them to edit before applying.

## When to Use

- Hook recommended `PreviewEnhancement` workflow
- User explicitly asks to "preview enhancement" or "show enhancement"
- Complex task where user should approve approach

## Steps

### Step 1: Gather Context

From the hook's system-reminder, extract:
- Classification type and confidence
- Generated enhancement (chain_of_thought, constraints)
- Complexity analysis
- Any gaps detected

### Step 2: Present Preview

Show the user what the enhanced prompt would look like:

```
Here's the proposed enhancement for your request:

**Original:** [user's original prompt]

**Type:** [classification.type] (confidence: [X]%)

**Enhancement:**
[Show the chain_of_thought steps]

[Show constraints if present]

Would you like to:
1. Apply as-is and proceed
2. Edit the enhancement first
3. Skip enhancement and proceed with original
```

### Step 3: Handle Response

**If user chooses "Apply as-is":**
- Proceed with the enhanced approach
- Apply the chain-of-thought structure to your work
- Keep constraints in mind

**If user chooses "Edit":**
- Ask what they'd like to change
- Incorporate their feedback
- Show updated preview
- Return to Step 3

**If user chooses "Skip":**
- Proceed with the original request
- No enhancement applied

### Step 4: Log (Optional)

If kai-history-system is available:
- Log the enhancement event
- Record user's choice
- Store for future learning

## Example Interaction

```
User: "fix the login bug"

Claude: Here's the proposed enhancement for your request:

**Original:** fix the login bug

**Type:** debug (confidence: 85%)

**Enhancement:**
<chain_of_thought>
1. Identify symptoms and error messages
2. Determine expected vs actual behavior
3. Trace code flow to find failure point
4. Identify root cause (not just symptoms)
5. Propose fix with verification steps
</chain_of_thought>

<constraints>
Success: Root cause identified and fixed
Failure: Symptom masked without understanding
</constraints>

Would you like to:
1. Apply as-is and proceed
2. Edit the enhancement first
3. Skip enhancement

User: 1

Claude: [Proceeds with debugging approach using the CoT structure]
```
```

#### 4.3: Create ComposeAgentTeam.md

```markdown
<!-- $PAI_DIR/Skills/InputEnhancer/Workflows/ComposeAgentTeam.md -->
# ComposeAgentTeam Workflow

Propose and customize a specialized agent team for complex multi-perspective tasks.

## When to Use

- Hook recommended `ComposeAgentTeam` workflow
- User explicitly asks for "agent team", "multi-agent", or "compose agents"
- Complex task requiring multiple perspectives (security audit, comprehensive review, research)

## Prerequisites

- **kai-agents-skill** must be installed for full functionality
- Without it, provide the analysis structure manually

## Steps

### Step 1: Analyze Task

From the hook's system-reminder:
- Note the `agentTeamType` suggestion (security, review, research, analysis)
- Review the complexity reasons
- Consider if plan mode is active (affects agent focus)

### Step 2: Load Team Profile

Load the appropriate team from `Data/AgentProfiles.yaml`:

```yaml
# Example: security-audit team
agents:
  - name: "Adversarial Tester"
    traits: ["security", "skeptical", "adversarial"]
    focus: "Find exploitable vulnerabilities"
  - name: "Security Architect"
    traits: ["security", "analytical", "systematic"]
    focus: "Recommend defenses and best practices"
  - name: "Compliance Auditor"
    traits: ["legal", "meticulous", "thorough"]
    focus: "Check regulatory requirements"
```

### Step 3: Present Team Proposal

Show the user the proposed team:

```
For this [task description], I recommend a [N]-agent team:

[Agent 1 Name] ([traits])
Focus: [focus area]

[Agent 2 Name] ([traits])
Focus: [focus area]

[Agent 3 Name] ([traits])
Focus: [focus area]

Would you like to:
1. Proceed with this team
2. Modify the team (add/remove/change agents)
3. Use a single agent instead
4. Skip agents and proceed manually
```

### Step 4: Handle Response

**If user chooses "Proceed":**
- Go to Step 5 (Launch Agents)

**If user chooses "Modify":**
- Ask what changes they want
- Options: add agent, remove agent, change traits, change focus
- Update team composition
- Return to Step 3

**If user chooses "Single agent":**
- Ask which perspective is most important
- Launch one agent with combined focus

**If user chooses "Skip":**
- Proceed without agents
- Apply enhancement manually

### Step 5: Launch Agents

**CRITICAL: Use `subagent_type: "general-purpose"` for custom agents!**

For each agent in the approved team:

1. **If kai-agents-skill is available:**
   ```bash
   bun run $PAI_DIR/Skills/Agents/Tools/AgentFactory.ts \
     --traits "[agent traits]" \
     --task "[task description]" \
     --output json
   ```

2. **Launch with Task tool:**
   ```
   Task({
     description: "[Agent Name]",
     prompt: [prompt from AgentFactory],
     subagent_type: "general-purpose",  // CRITICAL
     model: "sonnet"
   });
   ```

3. **Launch all agents in parallel** (single message with multiple Task calls)

### Step 6: Synthesize Results

Once all agents complete:
- Gather their outputs
- Identify common findings
- Note disagreements
- Synthesize into unified response
- Highlight key insights from each perspective

### Step 7: Log (Optional)

If kai-history-system is available:
- Log team composition
- Record agent outputs
- Store for future reference

## Example Interaction

```
User: "audit the API for security vulnerabilities"

Claude: For this security audit, I recommend a 3-agent team:

**Adversarial Tester** (security, skeptical, adversarial)
Focus: Find exploitable vulnerabilities

**Security Architect** (security, analytical, systematic)
Focus: Recommend defenses and best practices

**Compliance Auditor** (legal, meticulous, thorough)
Focus: Check regulatory requirements (OWASP, PCI-DSS)

Would you like to:
1. Proceed with this team
2. Modify the team
3. Single agent instead
4. Skip agents

User: 1

Claude: [Launches 3 agents in parallel]
[Waits for completion]
[Synthesizes findings from all three perspectives]
```
```

#### 4.4: Create AskClarification.md

```markdown
<!-- $PAI_DIR/Skills/InputEnhancer/Workflows/AskClarification.md -->
# AskClarification Workflow

Ask targeted questions to resolve ambiguity in user requests.

## When to Use

- Hook detected high-severity gaps (missing scope, target, goal)
- Classification confidence is low
- Multiple interpretations possible
- User explicitly asks to "clarify my request"

## Steps

### Step 1: Identify Gaps

From the hook's system-reminder, extract detected gaps:
- **scope**: Unclear what part of the system
- **target**: No specific file or component
- **goal**: Unclear success criteria
- **interpretation**: Multiple meanings possible

### Step 2: Load Question Patterns

Load from `Data/QuestionPatterns.yaml`:

```yaml
gaps:
  scope:
    question: "Which part of the system does this involve?"
    options_by_type:
      debug: ["Frontend", "Backend", "Database", "API", "Authentication"]
      feature: ["New component", "Existing feature", "Integration", "UI"]

  target:
    question: "What specific file or component?"
    followup: "Can you point me to the relevant code?"

  goal:
    question: "What's the main objective?"
    options_by_type:
      debug: ["Fix error", "Improve behavior", "Handle edge case"]
      refactor: ["Improve readability", "Reduce complexity", "Better patterns"]
```

### Step 3: Ask Questions

Present questions for each detected gap. Use the AskUserQuestion tool for structured input:

```
I want to make sure I understand your request correctly:

[For scope gap:]
Which part of the system does this involve?
- Frontend
- Backend
- Database
- API
- Other (please specify)

[For target gap:]
What specific file or component needs work?
[Free text input]

[For goal gap:]
What's the main objective?
- Fix an error
- Add a feature
- Improve performance
- Other (please specify)
```

### Step 4: Collect Answers

User responds through normal chat or AskUserQuestion selection.

### Step 5: Enrich Context

Build complete understanding from answers:
- Fill in the gaps
- Update classification if needed
- Re-assess complexity

### Step 6: Proceed

With enriched context, either:
- **Apply enhancement directly** (if now clear enough)
- **Show preview** (via PreviewEnhancement workflow)
- **Propose agents** (via ComposeAgentTeam workflow)

## Example Interaction

```
User: "fix it"

Claude: I want to make sure I understand your request. A few questions:

**Which part of the system needs fixing?**
- Frontend UI
- Backend API
- Database
- Authentication
- Other

**What's the symptom you're seeing?**
[Please describe the error or unexpected behavior]

User: "Backend API - getting 500 errors on /users endpoint"

Claude: Got it! You're seeing 500 errors on the /users endpoint in the backend API.

Let me debug this:
<chain_of_thought>
1. Identify the 500 error details from logs
2. Trace the /users endpoint handler
3. Check database connectivity and queries
4. Identify root cause
5. Propose fix
</chain_of_thought>

[Proceeds with debugging]
```

## Question Guidelines

1. **Be specific** - Ask about the exact gap, not general questions
2. **Provide options** - When possible, give choices to reduce friction
3. **Limit questions** - 2-3 max, don't overwhelm
4. **Allow free text** - Always include "Other" option
5. **Progress quickly** - Once answered, proceed without more questions
```

---

### Step 5: Create Data Files

#### 5.1: Create EnhancementTemplates.yaml

```yaml
# $PAI_DIR/Skills/InputEnhancer/Data/EnhancementTemplates.yaml
# Chain-of-thought templates by request type

templates:
  debug:
    cot: true
    chain_of_thought:
      - "Identify symptoms and error messages"
      - "Determine expected vs actual behavior"
      - "Trace code flow to find failure point"
      - "Identify root cause (not just symptoms)"
      - "Propose fix with verification steps"
    constraints:
      success: "Root cause identified and fixed"
      failure: "Symptom masked without understanding"
    plan_mode:
      chain_of_thought:
        - "What are all the reported symptoms?"
        - "What code paths could cause this behavior?"
        - "What similar bugs exist in the codebase history?"
        - "What testing gaps allowed this to occur?"
        - "What are the possible fix approaches and trade-offs?"
      constraints:
        success: "Bug fully understood with fix options documented"
        failure: "Jumping to fix without understanding"

  feature:
    cot: true
    chain_of_thought:
      - "Clarify requirements and acceptance criteria"
      - "Identify where in codebase this belongs"
      - "Consider edge cases and error handling"
      - "Implement with minimal complexity"
      - "Verify feature works as intended"
    constraints:
      success: "Feature implemented and tested"
      failure: "Over-engineered or missing requirements"
    plan_mode:
      chain_of_thought:
        - "What exactly should this feature do?"
        - "How do similar features work in this codebase?"
        - "What are the architectural options?"
        - "What dependencies and integrations are needed?"
        - "What are the risks and unknowns?"
      constraints:
        success: "Feature fully scoped with clear implementation plan"
        failure: "Starting implementation without understanding scope"

  explain:
    cot: false
    structure:
      - "High-level overview"
      - "Key components and relationships"
      - "Concrete examples"
      - "Summary of key points"

  refactor:
    cot: true
    chain_of_thought:
      - "Identify code smells and issues"
      - "Determine target structure"
      - "Plan incremental changes"
      - "Implement with tests passing"
      - "Verify behavior unchanged"
    constraints:
      success: "Code improved without breaking functionality"
      failure: "Introduced bugs or broke existing features"
    plan_mode:
      chain_of_thought:
        - "What specific issues need addressing?"
        - "What patterns exist in similar code?"
        - "What are the refactoring options?"
        - "What risks does each approach carry?"
        - "How to verify nothing breaks?"
      constraints:
        success: "Refactoring plan with clear steps and verification"
        failure: "Starting changes without understanding impact"

  research:
    cot: true
    chain_of_thought:
      - "Define research questions clearly"
      - "Identify reliable sources"
      - "Gather diverse perspectives"
      - "Synthesize findings"
      - "Provide actionable recommendations"
    constraints:
      success: "Comprehensive understanding with clear next steps"
      failure: "Surface-level findings without depth"

  review:
    cot: true
    chain_of_thought:
      - "Understand the scope and purpose"
      - "Check for bugs and edge cases"
      - "Evaluate design and patterns"
      - "Consider security implications"
      - "Provide constructive feedback"
    constraints:
      success: "Thorough review with actionable feedback"
      failure: "Superficial check missing important issues"
    plan_mode:
      chain_of_thought:
        - "What is the scope of this review?"
        - "What standards and patterns apply?"
        - "What are the key risk areas?"
        - "What review checklist should be used?"
        - "How to prioritize findings?"
      constraints:
        success: "Review strategy defined with clear criteria"
        failure: "Starting review without clear focus"

  test:
    cot: true
    chain_of_thought:
      - "Identify what needs testing"
      - "Define test cases and edge cases"
      - "Set up test fixtures"
      - "Implement tests"
      - "Verify coverage"
    constraints:
      success: "Comprehensive test coverage"
      failure: "Tests that pass but miss important cases"
    plan_mode:
      chain_of_thought:
        - "What behavior needs to be tested?"
        - "What are the edge cases and failure modes?"
        - "What testing strategy fits best?"
        - "What mocking or fixtures are needed?"
        - "How to measure coverage?"
      constraints:
        success: "Test plan with clear coverage goals"
        failure: "Starting tests without understanding what to cover"

  document:
    cot: false
    structure:
      - "Purpose and overview"
      - "Installation/setup"
      - "Usage examples"
      - "API reference (if applicable)"
      - "Troubleshooting"

  deploy:
    cot: true
    chain_of_thought:
      - "Verify code is ready for deployment"
      - "Check environment configuration"
      - "Plan rollback strategy"
      - "Execute deployment"
      - "Verify deployment success"
    constraints:
      success: "Successful deployment with verification"
      failure: "Deployment issues or missing rollback plan"
    plan_mode:
      chain_of_thought:
        - "What needs to be deployed?"
        - "What environments are involved?"
        - "What are the dependencies?"
        - "What could go wrong?"
        - "What's the rollback plan?"
      constraints:
        success: "Deployment plan with clear steps and rollback"
        failure: "Deploying without understanding risks"

  config:
    cot: true
    chain_of_thought:
      - "Understand configuration requirements"
      - "Identify affected components"
      - "Make changes incrementally"
      - "Test configuration"
      - "Document changes"
    constraints:
      success: "Configuration working and documented"
      failure: "Misconfiguration causing runtime issues"

  general:
    cot: false
    structure:
      - "Understand the request"
      - "Break into steps if needed"
      - "Execute methodically"
      - "Verify completion"
```

#### 5.2: Create AgentProfiles.yaml

```yaml
# $PAI_DIR/Skills/InputEnhancer/Data/AgentProfiles.yaml
# Pre-defined agent teams for complex tasks

teams:
  security:
    description: "Multi-perspective security analysis"
    when: "Security audits, vulnerability assessments, penetration testing"
    agents:
      - name: "Adversarial Tester"
        traits: ["security", "skeptical", "adversarial"]
        focus: "Find exploitable vulnerabilities"
        questions:
          - "How could an attacker abuse this?"
          - "What happens with malformed input?"
          - "Are there any authentication bypasses?"

      - name: "Security Architect"
        traits: ["security", "analytical", "systematic"]
        focus: "Recommend defenses and best practices"
        questions:
          - "What security patterns should be applied?"
          - "How should data be protected?"
          - "What's the defense in depth strategy?"

      - name: "Compliance Auditor"
        traits: ["legal", "meticulous", "thorough"]
        focus: "Check regulatory requirements"
        questions:
          - "Does this meet OWASP Top 10 requirements?"
          - "Are there PCI-DSS implications?"
          - "What audit trail is needed?"

  review:
    description: "Comprehensive code quality review"
    when: "Code reviews, PR reviews, quality assessments"
    agents:
      - name: "Bug Hunter"
        traits: ["technical", "skeptical", "thorough"]
        focus: "Find bugs and edge cases"
        questions:
          - "What happens at boundary conditions?"
          - "Are there race conditions?"
          - "What if this fails?"

      - name: "Architecture Reviewer"
        traits: ["technical", "analytical", "systematic"]
        focus: "Evaluate design patterns and structure"
        questions:
          - "Does this follow existing patterns?"
          - "Is the abstraction level appropriate?"
          - "How does this affect maintainability?"

      - name: "Performance Analyst"
        traits: ["technical", "meticulous", "comparative"]
        focus: "Identify performance bottlenecks"
        questions:
          - "What's the time/space complexity?"
          - "Are there N+1 queries?"
          - "How does this scale?"

  research:
    description: "Multi-perspective research synthesis"
    when: "Research tasks, technology evaluation, decision-making"
    agents:
      - name: "Explorer"
        traits: ["research", "enthusiastic", "exploratory"]
        focus: "Find diverse sources and options"
        questions:
          - "What are all the alternatives?"
          - "What does the community prefer?"
          - "What's cutting edge?"

      - name: "Critic"
        traits: ["research", "skeptical", "analytical"]
        focus: "Evaluate credibility and downsides"
        questions:
          - "What are the hidden costs?"
          - "What could go wrong?"
          - "Is this source reliable?"

      - name: "Synthesizer"
        traits: ["research", "analytical", "synthesizing"]
        focus: "Combine insights into recommendations"
        questions:
          - "What's the common thread?"
          - "What trade-offs exist?"
          - "What's the recommendation?"

  analysis:
    description: "Multi-perspective analysis"
    when: "Architecture decisions, trade-off analysis, complex problems"
    agents:
      - name: "Pragmatist"
        traits: ["technical", "practical", "efficient"]
        focus: "Find the simplest solution"
        questions:
          - "What's the minimum viable approach?"
          - "What can we defer?"
          - "What's the fastest path?"

      - name: "Perfectionist"
        traits: ["technical", "meticulous", "thorough"]
        focus: "Ensure quality and completeness"
        questions:
          - "What edge cases are we missing?"
          - "Is this the right abstraction?"
          - "Will this scale?"

      - name: "Devil's Advocate"
        traits: ["analytical", "skeptical", "contrarian"]
        focus: "Challenge assumptions"
        questions:
          - "Why not do the opposite?"
          - "What if our assumptions are wrong?"
          - "What are we not seeing?"
```

#### 5.3: Create QuestionPatterns.yaml

```yaml
# $PAI_DIR/Skills/InputEnhancer/Data/QuestionPatterns.yaml
# Question patterns for clarification by gap type

gaps:
  scope:
    question: "Which part of the system does this involve?"
    severity: high
    options_by_type:
      debug:
        - "Frontend UI"
        - "Backend API"
        - "Database"
        - "Authentication"
        - "Third-party integration"
      feature:
        - "New standalone component"
        - "Extension of existing feature"
        - "Integration between systems"
        - "UI/UX improvement"
      refactor:
        - "Single file/function"
        - "Module/package"
        - "Cross-cutting concern"
        - "Architecture-level"
      review:
        - "Specific file"
        - "Pull request"
        - "Entire codebase"
        - "Specific concern (security, performance)"
    fallback_options:
      - "Frontend"
      - "Backend"
      - "Database"
      - "Infrastructure"
      - "Other"

  target:
    question: "What specific file or component?"
    severity: medium
    followup: "Can you point me to the relevant code or describe where to find it?"
    examples:
      - "src/auth/login.ts"
      - "The user registration flow"
      - "The API endpoint for /users"

  goal:
    question: "What's the main objective?"
    severity: high
    options_by_type:
      debug:
        - "Fix a specific error"
        - "Improve reliability"
        - "Handle edge case"
        - "Understand why it fails"
      feature:
        - "Add new functionality"
        - "Enhance existing feature"
        - "Integrate with external system"
        - "Improve user experience"
      refactor:
        - "Improve readability"
        - "Reduce complexity"
        - "Apply better patterns"
        - "Improve performance"
      review:
        - "Find bugs"
        - "Check security"
        - "Evaluate architecture"
        - "Ensure standards compliance"
    fallback_options:
      - "Fix something broken"
      - "Add something new"
      - "Improve something existing"
      - "Understand something"

  interpretation:
    question: "I want to make sure I understand. Do you mean:"
    severity: low
    note: "Options are generated dynamically based on possible interpretations"
    example_prompt: "update the user"
    example_options:
      - "Update the user model/schema"
      - "Update user data in the database"
      - "Update the user interface"
      - "Update user documentation"

# General clarification settings
settings:
  max_questions: 3
  always_include_other: true
  preferred_format: "multiple_choice"
  fallback_format: "free_text"
```

---

### Step 6: Create Tool

#### 6.1: Create GenerateEnhancement.ts

```typescript
// $PAI_DIR/Skills/InputEnhancer/Tools/GenerateEnhancement.ts
// CLI tool for generating enhancements

import { parseArgs } from 'util';
import { classifyPrompt } from '../../../hooks/lib/input-enhancer/classifier';
import { analyzeComplexity } from '../../../hooks/lib/input-enhancer/complexity-detector';
import { detectPlanMode } from '../../../hooks/lib/input-enhancer/plan-mode-detector';
import { generateEnhancement, formatEnhancementXml } from '../../../hooks/lib/input-enhancer/enhancement-generator';

const HELP = `
GenerateEnhancement - Generate enhancement for a prompt

USAGE:
  bun run GenerateEnhancement.ts --prompt "your prompt here"
  echo "your prompt" | bun run GenerateEnhancement.ts

OPTIONS:
  --prompt, -p    The prompt to enhance
  --type, -t      Force classification type (debug, feature, explain, etc.)
  --plan-mode     Force plan mode (true/false)
  --format, -f    Output format: text (default), json, xml
  --help, -h      Show this help

EXAMPLES:
  # Basic usage
  bun run GenerateEnhancement.ts -p "fix the login bug"

  # Force type
  bun run GenerateEnhancement.ts -p "check the code" -t review

  # JSON output
  bun run GenerateEnhancement.ts -p "add user auth" -f json

  # Pipe input
  echo "explain the auth middleware" | bun run GenerateEnhancement.ts
`;

async function main() {
  const { values, positionals } = parseArgs({
    args: Bun.argv.slice(2),
    options: {
      prompt: { type: 'string', short: 'p' },
      type: { type: 'string', short: 't' },
      'plan-mode': { type: 'string' },
      format: { type: 'string', short: 'f', default: 'text' },
      help: { type: 'boolean', short: 'h' }
    },
    allowPositionals: true
  });

  if (values.help) {
    console.log(HELP);
    process.exit(0);
  }

  // Get prompt from args or stdin
  let prompt = values.prompt || positionals.join(' ');

  if (!prompt) {
    // Try reading from stdin
    const stdin = await Bun.stdin.text();
    prompt = stdin.trim();
  }

  if (!prompt) {
    console.error('Error: No prompt provided. Use --prompt or pipe input.');
    console.error('Use --help for usage information.');
    process.exit(1);
  }

  // Override plan mode if specified
  if (values['plan-mode']) {
    process.env.CLAUDE_PLAN_MODE = values['plan-mode'];
  }

  // Classify
  let classification = classifyPrompt(prompt);

  // Override type if specified
  if (values.type) {
    classification = {
      ...classification,
      type: values.type as any,
      matchedPatterns: [...classification.matchedPatterns, `forced:${values.type}`]
    };
  }

  // Detect plan mode
  const planMode = await detectPlanMode();

  // Analyze complexity
  const complexity = analyzeComplexity(prompt, classification);

  // Generate enhancement
  const result = await generateEnhancement(prompt, classification, complexity, planMode);

  // Output based on format
  switch (values.format) {
    case 'json':
      console.log(JSON.stringify(result, null, 2));
      break;

    case 'xml':
      console.log(formatEnhancementXml(result));
      break;

    case 'text':
    default:
      console.log('=== Input Enhancement ===\n');
      console.log(`Original: ${prompt}\n`);
      console.log(`Type: ${result.classification.type}`);
      console.log(`Confidence: ${result.classification.confidence}%`);
      console.log(`Complexity: ${result.complexity.level}`);
      console.log(`Plan Mode: ${result.planMode.active ? 'Active' : 'Inactive'}`);
      console.log('');

      if (result.complexity.gaps.length > 0) {
        console.log('Gaps Detected:');
        for (const gap of result.complexity.gaps) {
          console.log(`  - [${gap.severity}] ${gap.type}: ${gap.description}`);
        }
        console.log('');
      }

      console.log('Recommendation:');
      console.log(`  Action: ${result.recommendation.action}`);
      if (result.recommendation.workflow) {
        console.log(`  Workflow: ${result.recommendation.workflow}`);
      }
      console.log(`  Reason: ${result.recommendation.reason}`);
      console.log('');

      if (result.enhancement.chainOfThought) {
        console.log('Chain of Thought:');
        result.enhancement.chainOfThought.forEach((step, i) => {
          console.log(`  ${i + 1}. ${step}`);
        });
        console.log('');
      }

      if (result.enhancement.constraints) {
        console.log('Constraints:');
        console.log(`  Success: ${result.enhancement.constraints.success}`);
        console.log(`  Failure: ${result.enhancement.constraints.failure}`);
      }
      break;
  }
}

main().catch(console.error);
```

---

### Step 7: Register Hook in settings.json

Claude Code looks for settings in `~/.claude/settings.json`. Add or merge the following hook configuration:

**File location:** `~/.claude/settings.json`

**If you have no existing hooks:**

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bun run $PAI_DIR/hooks/input-enhancer.ts"
          }
        ]
      }
    ]
  }
}
```

**If you already have UserPromptSubmit hooks (MERGE):**

Add the input-enhancer hook to your existing array:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "bun run $PAI_DIR/hooks/existing-hook.ts"
          },
          {
            "type": "command",
            "command": "bun run $PAI_DIR/hooks/input-enhancer.ts"
          }
        ]
      }
    ]
  }
}
```

---

### Step 8: Verify Installation

```bash
PAI_DIR="${PAI_DIR:-$HOME/.config/pai}"

# 1. Check hook exists
echo "1. Checking hook file..."
ls -la "$PAI_DIR/hooks/input-enhancer.ts"

# 2. Check lib files exist
echo ""
echo "2. Checking library files..."
ls -la "$PAI_DIR/hooks/lib/input-enhancer/"

# 3. Check skill exists
echo ""
echo "3. Checking skill files..."
ls -la "$PAI_DIR/Skills/InputEnhancer/"

# 4. Check workflows exist
echo ""
echo "4. Checking workflows..."
ls -la "$PAI_DIR/Skills/InputEnhancer/Workflows/"

# 5. Check data files exist
echo ""
echo "5. Checking data files..."
ls -la "$PAI_DIR/Skills/InputEnhancer/Data/"

# 6. Check tool exists
echo ""
echo "6. Checking tool..."
ls -la "$PAI_DIR/Skills/InputEnhancer/Tools/"

# 7. Test high-clarity input (should output classification)
echo ""
echo "7. Testing high-clarity input..."
echo '{"session_id":"test","prompt":"Fix TypeError in auth.ts line 45"}' | \
  bun run "$PAI_DIR/hooks/input-enhancer.ts"
echo ""
echo "Expected: <system-reminder> with Type: debug, Confidence: 90%+"

# 8. Test complex input (should suggest agents)
echo ""
echo "8. Testing complex input..."
echo '{"session_id":"test","prompt":"audit the payment system security"}' | \
  bun run "$PAI_DIR/hooks/input-enhancer.ts"
echo ""
echo "Expected: Complexity: HIGH, Suggestion: ComposeAgentTeam"

# 9. Test bypass
echo ""
echo "9. Testing bypass prefix..."
echo '{"session_id":"test","prompt":"!raw: just do it"}' | \
  bun run "$PAI_DIR/hooks/input-enhancer.ts"
echo ""
echo "Expected: (no output - bypassed)"

# 10. Test short input
echo ""
echo "10. Testing short input..."
echo '{"session_id":"test","prompt":"yes"}' | \
  bun run "$PAI_DIR/hooks/input-enhancer.ts"
echo ""
echo "Expected: (no output - skipped)"

# 11. Test CLI tool
echo ""
echo "11. Testing CLI tool..."
bun run "$PAI_DIR/Skills/InputEnhancer/Tools/GenerateEnhancement.ts" \
  --prompt "fix the login bug" --format text
```

---

## Invocation Scenarios

The input enhancer triggers automatically on every user input:

| Scenario | Hook Output | Claude Action | User Experience |
|----------|-------------|---------------|-----------------|
| Short input (<10 chars) | None (skipped) | Normal processing | No change |
| Bypass prefix (`!raw:`) | None (bypassed) | Normal processing | No change |
| Simple + high confidence | Classification + light CoT | Apply directly | Slight delay, better structure |
| Complex task | Classification + agent suggestion | May invoke skill | Asked about agent team |
| Ambiguous input | Classification + gaps | May invoke skill | Asked clarifying questions |
| Plan mode active | Research-focused CoT | Research approach | Planning-appropriate guidance |

### Trigger Examples

| Input | Type | Confidence | Complexity | Action |
|-------|------|------------|------------|--------|
| "yes" | - | - | - | Skip (too short) |
| "!raw: do it now" | - | - | - | Bypass (prefix) |
| "fix the bug in auth.ts" | debug | 85% | simple | Apply direct |
| "explain the auth flow" | explain | 80% | simple | Apply direct |
| "audit API security" | review | 87% | complex | Suggest agents |
| "update it" | general | 30% | moderate | Ask clarification |
| "add user auth" (in plan mode) | feature | 75% | moderate | Research-focused CoT |

---

## Example Usage

### Example 1: Simple Debug Task

```
User: "fix the TypeError in auth.ts line 45"

[Hook outputs system-reminder]

<system-reminder>
[InputEnhancer] Analysis Complete

Type: debug
Confidence: 92%
Complexity: SIMPLE

Enhancement Preview:
<enhancement>
<chain_of_thought>
1. Identify symptoms and error messages
2. Determine expected vs actual behavior
3. Trace code flow to find failure point
4. Identify root cause (not just symptoms)
5. Propose fix with verification steps
</chain_of_thought>

<constraints>
Success: Root cause identified and fixed
Failure: Symptom masked without understanding
</constraints>
</enhancement>

Action: Apply enhancement directly and proceed with the task.
</system-reminder>

Claude: I'll debug the TypeError in auth.ts line 45. Following a systematic approach:

1. First, let me read the file and examine the error context...
[Proceeds with structured debugging]
```

### Example 2: Complex Security Audit

```
User: "audit the API for security vulnerabilities"

[Hook outputs system-reminder]

<system-reminder>
[InputEnhancer] Analysis Complete

Type: review
Confidence: 87%
Complexity: HIGH

Agent Team Suggested: security
Team to do the work

Recommendation:
- Action: invoke skill
- Suggested workflow: ComposeAgentTeam
- Reason: Complex task would benefit from security agent team
</system-reminder>

Claude: This is a security audit that would benefit from multiple perspectives.
I recommend a 3-agent team:

**Adversarial Tester** (security, skeptical, adversarial)
Focus: Find exploitable vulnerabilities

**Security Architect** (security, analytical, systematic)
Focus: Recommend defenses and best practices

**Compliance Auditor** (legal, meticulous, thorough)
Focus: Check regulatory requirements (OWASP, PCI-DSS)

Would you like to:
1. Proceed with this team
2. Modify the team
3. Single agent instead
4. Skip agents

User: 1

Claude: [Launches 3 agents in parallel]
[Each agent analyzes with their perspective]
[Synthesizes findings]

Security Audit Complete:

**Adversarial Tester found:**
- SQL injection in /users endpoint
- Missing rate limiting on /auth
- CORS misconfiguration

**Security Architect recommends:**
- Parameterized queries for all DB access
- Rate limiting middleware
- Strict CORS policy

**Compliance notes:**
- Meets OWASP A03 (Injection) after fix
- Needs audit logging for PCI-DSS
```

### Example 3: Ambiguous Input

```
User: "fix it"

[Hook outputs system-reminder with gaps detected]

<system-reminder>
[InputEnhancer] Analysis Complete

Type: debug
Confidence: 45%
Complexity: MODERATE

Gaps Detected:
- [HIGH] scope: Missing scope - unclear what part of the system is involved
- [HIGH] target: Missing target - no specific file or component mentioned
- [HIGH] goal: Missing goal - unclear what success looks like

Recommendation:
- Action: invoke skill
- Suggested workflow: AskClarification
- Reason: High severity gaps detected - need clarification
</system-reminder>

Claude: I want to make sure I understand your request. A few questions:

**Which part of the system needs fixing?**
- Frontend UI
- Backend API
- Database
- Authentication
- Other

**What's the symptom you're seeing?**
[Please describe the error or behavior]

User: Backend API - getting 500 errors on /users endpoint

Claude: Got it! You're seeing 500 errors on the /users endpoint.

<chain_of_thought>
1. Check error logs for the 500 errors
2. Trace the /users endpoint handler
3. Identify what's causing the server error
4. Fix root cause
5. Verify endpoint works
</chain_of_thought>

Let me investigate...
```

### Example 4: Plan Mode Active

```
User: "add user authentication" (while plan mode is active)

[Hook detects plan mode, adjusts enhancement]

<system-reminder>
[InputEnhancer] Analysis Complete

Mode: PLAN MODE (exploring)
Focus: research and exploration

Type: feature
Confidence: 85%
Complexity: MODERATE

Enhancement Preview:
<enhancement>
<chain_of_thought>
1. What exactly should this feature do?
2. How do similar features work in this codebase?
3. What are the architectural options?
4. What dependencies and integrations are needed?
5. What are the risks and unknowns?
</chain_of_thought>

<constraints>
Success: Feature fully scoped with clear implementation plan
Failure: Starting implementation without understanding scope
</constraints>
</enhancement>

Action: Apply enhancement directly and proceed with the task.
</system-reminder>

Claude: Since we're in planning mode, let me explore the authentication landscape before we design the implementation.

First, I'll research:
1. What authentication patterns exist in this codebase?
2. What are the architectural options (JWT, session, OAuth)?
3. What dependencies are already available?
4. What risks should we consider?

[Explores codebase]
[Documents findings to plan file]
[Proposes options for user to choose]
```

---

## Configuration

**Environment variables:**

**Option 1: `.env` file** (recommended - created by Kai Bundle wizard):
```bash
# $PAI_DIR/.env
PAI_DIR="$HOME/.config/pai"
PAI_INPUT_ENHANCE="true"  # Set to "false" to disable enhancement
PAI_DEBUG="false"         # Set to "true" for debug logging
```

**Option 2: Shell profile** (for manual installation):
```bash
# Add to ~/.zshrc or ~/.bashrc
export PAI_DIR="$HOME/.config/pai"
export PAI_INPUT_ENHANCE="true"
export PAI_DEBUG="false"
```

**Bypass mechanisms:**
- Prefix with `!raw:` to bypass enhancement
- Prefix with `*` to bypass enhancement
- Environment `PAI_INPUT_ENHANCE=false` disables globally

---

## Customization

### Recommended Customization

**What to Customize:** Enhancement templates in `EnhancementTemplates.yaml`

**Why:** Different teams and projects have different workflows. Customize the chain-of-thought steps to match your actual development process.

**Process:**
1. Open `$PAI_DIR/Skills/InputEnhancer/Data/EnhancementTemplates.yaml`
2. Review the existing templates for each type
3. Modify the `chain_of_thought` steps to match your workflow
4. Add project-specific constraints
5. Test with the CLI tool

**Expected Outcome:** Enhancements that match your team's actual development practices.

---

### Optional Customization

| Customization | File | Impact |
|--------------|------|--------|
| Add new request types | `classifier.ts` | New classification categories |
| Custom agent teams | `AgentProfiles.yaml` | Project-specific agent compositions |
| Different questions | `QuestionPatterns.yaml` | Clarification flow |
| Complexity thresholds | `complexity-detector.ts` | When agents are suggested |
| Plan mode detection | `plan-mode-detector.ts` | How plan mode is detected |

---

## Credits

- **Original concept**: Daniel Miessler - developed as part of Kai personal AI infrastructure
- **Inspired by**: Anthropic's Prompt Improver, DSPy framework, severity1's claude-code-prompt-improver
- **Chain-of-thought techniques**: Based on Anthropic's research on structured prompting

---

## Related Work

- [Anthropic Prompt Improver](https://docs.anthropic.com/claude/page/prompt-improver) - Official prompt enhancement
- [DSPy](https://github.com/stanfordnlp/dspy) - Framework for programming with language models
- [severity1/claude-code-prompt-improver](https://github.com/severity1/claude-code-prompt-improver) - Similar hook for Claude Code

---

## Works Well With

- **kai-agents-skill** - For multi-agent composition features
- **kai-prompting-skill** - Shares Standards.md principles
- **kai-history-system** - For logging enhancements

---

## Recommended

- **kai-agents-skill** - Highly recommended for complex multi-perspective tasks
- **kai-history-system** - Recommended for tracking enhancement effectiveness

---

## Relationships

### Parent Of
*None specified.*

### Child Of
- **kai-hook-system** - Depends on UserPromptSubmit hook infrastructure

### Sibling Of
- **kai-prompting-skill** - Complementary prompting capabilities

### Part Of Collection
- **Kai Bundle** - Part of the Kai personal AI infrastructure

---

## Changelog

### 1.0.0 - 2025-12-30
- Initial release
- Two-part architecture (hook + skill)
- 11 request type classifications
- Chain-of-thought templates
- Plan mode detection and adjustment
- Multi-agent composition via skill
- Preview and edit workflow
- Clarification question workflow
- Integration with kai-agents-skill
