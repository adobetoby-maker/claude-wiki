# Category: Language Learning App

Used by: language-lens-elite (LinguaLens), juniorlinguist, language-threshold, medicalspanish, constructionspanish, future learning apps.
Compose with another category for stack specifics.

## Definition

Apps where the primary value is helping a user acquire language skill. Distinguished by:

- Repetition + spacing matter (SRS / SM-2 algorithms)
- Multi-modal input/output (text, audio, sometimes camera)
- Per-user progress tracking and gamification
- AI-augmented features (tutor, grammar explainer, pronunciation feedback)

## Stack Choices Used in This Category

| Project | Frontend | Runtime | Notes |
|---|---|---|---|
| language-lens-elite (LinguaLens) | TanStack Start + React Router v7 + Vite | Cloudflare Workers (via @cloudflare/vite-plugin) | Not Next.js |
| juniorlinguist | Vite + React | Vercel | Kids-facing, large fonts |
| language-threshold | Next.js 16 | Cloudflare Pages | Adult professional learners |
| medicalspanish, constructionspanish | Next.js 16 | Cloudflare Pages | Vertical-specific |

## Core Domain Concepts

### Levels
CEFR: A1 → A2 → B1 → B2 → C1 → C2. Always store level as a string enum, never an integer.

### XP and Tiers
Two separate tier systems — don't conflate:

**XP tier** (progression by total study): Beginner → Apprentice → Scholar → Linguist → Maestro
**Rank tier** (matchmaking / competitive): Bronze → Silver → Gold → Platinum → Diamond → Champion → Unreal

Store both. Derive from a single source of truth on every update.

### SRS (Spaced Repetition)
Default to SM-2 algorithm for review scheduling. Each card has:
- `interval` (days)
- `ease_factor` (default 2.5)
- `repetitions` (consecutive correct)
- `due_date`

Quality 0–5 input adjusts ease and interval. Failed reviews reset interval.

### Streak
Consecutive days of any qualifying activity. Resets on a missed day. UTC-based to avoid timezone gaming.

## Required Storage Schema (Supabase or D1)

Minimum tables for any learning app:

```sql
profiles (user_id, xp, xp_tier, streak, last_active, level, target_lang)
progress (user_id, item_id, repetitions, interval, ease, due_date)
sessions (user_id, started_at, duration, items_studied, xp_earned)
achievements (user_id, achievement_id, earned_at)
```

Project-specific tables add on top.

## AI Integration Patterns

### Server-side only
Never expose Anthropic/OpenAI keys to the client. AI calls go through server functions or Route Handlers.

```typescript
// TanStack Start example (language-lens-elite)
import { createServerFn } from '@tanstack/start'

export const explainGrammar = createServerFn('POST', async ({ sentence, lang }) => {
  const { Anthropic } = await import('@anthropic-ai/sdk')
  // ...
})
```

### Validate every input with Zod
User input flows directly into LLM prompts. Always validate:

```typescript
const schema = z.object({
  sentence: z.string().min(1).max(500),
  lang: z.enum(['ja', 'ko', 'es', 'fr', 'de']),
})
```

### Pick the right model
- **Vocabulary lookup, kana/hangul conversion** → Haiku (fast, cheap, deterministic)
- **Grammar explanation, translation, examples** → Haiku for short, Sonnet for complex
- **Tutor conversation, essay feedback** → Sonnet
- **Difficult linguistic edge cases** → Sonnet with extended thinking

See ~/.claude/decisions/0005-model-routing.md.

### Caching
For deterministic lookups (kana conversion, common phrases), cache aggressively. Same input → same output, no reason to recompute.

```typescript
// Use KV (Cloudflare) or a simple Supabase table
const cached = await kv.get(`kana:${input}`)
if (cached) return cached
```

## Multi-Script Rendering

Each writing system has gotchas:

### Japanese
- Furigana: ruby annotations. Three render modes: `off` | `above` (float) | `inline` (overlay)
- Persist mode in localStorage per project (key format: `<project>.reader.furigana.v1`)
- Use `FuriganaText.tsx` component pattern from language-lens-elite

### Korean
- Hangul + romaja toggle (same three modes as furigana)
- `HangulText.tsx` component pattern
- Romanization differs by system (Revised vs McCune-Reischauer) — pick one, document it

### Romance languages
- Diacritics matter for search (café ≠ cafe). Normalize with `String.normalize('NFD')` for search indexes only — never display the normalized form.

### Mandarin (if added)
- Pinyin + tones above characters (similar to furigana)
- Simplified vs Traditional — pick one per user setting

## Tokenization Pattern

`ClickableText.tsx` handles word-level click-to-lookup. Each language needs a tokenizer:

- Japanese: kuromoji.js or server-side via Claude
- Korean: jamo or syllable splitting
- Chinese: jieba (server) or character-by-character
- European: split on whitespace + punctuation

Click handler opens a word card with definition, examples, audio. Cache lookups.

## Audio (TTS)

Default providers:
- **Free / open-source**: Cloudflare Workers AI (multi-language) for short utterances
- **Quality**: ElevenLabs for tutor voices and long passages
- **Browser fallback**: Web Speech API (`speechSynthesis`) — works offline but voice quality varies

Always prefetch audio for active session items.

## Speech Recognition (STT)

- **Browser-native**: Web Speech API — fastest, no network round-trip, limited language support
- **Whisper**: via OpenAI or self-hosted (whisper.cpp on Workers AI) — accurate, slower
- **For pronunciation feedback**: forced alignment with a phoneme model (e.g. Whisper + alignment via gentle)

## Gamification Patterns

| Element | When to use |
|---|---|
| XP points | Always — primary engagement loop |
| Streak | Always — daily-return driver |
| Achievements | After 5+ sessions exist to anchor against |
| Leaderboards | Weekly/monthly, never lifetime (demoralizing for new users) |
| Tier promotions | Visual celebration animation, not a toast |
| Loss-aversion (streak freeze) | Optional, but increases retention 15–30% |

Don't add gamification before you have content. It's a multiplier, not a substitute.

## State Architecture

Per-domain context providers, not one global store. From language-lens-elite:

```
AuthProvider
  → AppProvider (XP, streak, tier — global learner state)
    → LibraryProvider
      → NotesProvider
        → GrammarProvider
          → SpeechProvider
            → SpeakProvider
              → TutorProvider
                → MatchProvider
                  → LeaderboardProvider
```

Each owns its slice. Persist what matters to Supabase, keep ephemeral state local.

## Tab/Screen Registry Pattern

If the app has tabs or screens, use an exhaustive `Record<TabKey, ComponentType>` map. TypeScript enforces no missed updates:

```typescript
// src/components/tab-registry.ts
export type TabKey = 'reader' | 'grammar' | 'speak' | 'kana' | ...
export const TAB_COMPONENTS: Record<TabKey, ComponentType> = {
  reader: ParallelReader,
  grammar: Grammar,
  // ...
}
```

Adding a tab requires updating both the union AND the map. TS catches the second miss.

## Failure Modes

- Adding a tab to one place but not the other → silent runtime failure
- Tokenizing client-side for CJK without considering bundle size → 500kb+ on first load
- Caching TTS audio in memory instead of persistent storage → wasted regenerations on every session
- Treating XP tier and rank tier as the same value → corrupted progress display
- Streak counter in user's local timezone → easy to game by setting clock back
- Triggering achievement animation inside a `useEffect` without a dependency guard → fires repeatedly
- Asking the LLM for free-form output that needs parsing → use structured output (JSON schema or `tool_use`)
- Not validating user input before sending to LLM → prompt injection risk

## Pedagogy Patterns Worth Encoding

| Pattern | What it does |
|---|---|
| Spaced repetition | Schedule next review based on recall confidence |
| Comprehensible input + 1 (i+1) | Show content slightly above current level |
| Interleaving | Mix related concepts rather than blocking by topic |
| Retrieval practice | Test recall, not just exposure |
| Output > input | Speaking/writing produces stronger encoding than reading/listening |
| Spacing > massing | 15 min daily beats 2 hours weekly |

Use these to inform feature priorities. A "review streak" beats a "study time" leaderboard.

## Decision Defaults

| User says | Default |
|---|---|
| "add a language" | Tokenizer + TTS + display component, in that order |
| "add a tab/screen" | Update TabKey union AND registry map together |
| "add SRS" | SM-2 unless otherwise specified |
| "translate this" | Server function with Zod-validated input |
| "fix audio" | Check fallback chain: Workers AI → ElevenLabs → browser native |
| "user accounts" | Supabase auth, never roll your own |
