# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

デイリー 生成AIクイズ — 1日3問の4択（90問を30日で1周）。`index.html` 1枚（ビルド無し・依存なし・音なし）で、
GitHub Pages（`main` から自動公開）: https://masa0980-sudo.github.io/daily-genai-quiz/ 。
**姉妹作 `daily-claude-quiz` とコードはほぼ同一**（問題データ `QUIZ`・文言・localStorage のキーだけが違う）。
共通化していないので、ロジックを直すときは**2か所とも**直すこと。

## Mini mode (`?mini=1`, called from mini-medley)

The hub in `C:\claude\repos\mini-medley` opens this page in an iframe with `?mini=1&sec=20&mute=0|1`.
**The protocol is specified in mini-medley's CLAUDE.md**; this section only records the quiz's side.
Without `mini=1` everything behaves exactly as before (regression-checked: start → 3 questions → result, history saved).

- A one-line `<head>` script adds `<html class="mini">`; CSS hides `#title`, `#result`, `#review`, `#nextBtn` and shows `#quiz`.
- It does **not** use the daily 3 questions: `miniGo()` shuffles `QUIZ` and takes 3 random ones, so repeated plays differ.
- Flow: `miniBoot()` posts `mini-ready` and waits → hub's `mini-go` → `miniShow(k)` per question with a time limit
  `MINI_PER = (sec − 0.9×3) / 3` (≈5.8 s at `sec=20`, shown by `#miniBar`); answering (or time-out = wrong) shows
  right/wrong for 0.9 s then **auto-advances** (no explanation, no "next" button) → `miniEnd()` posts `mini-result`.
- Score: each correct answer is worth 20 to 33.3 points depending on how much of the question's time was left; wrong or
  timed out = 0; total clamped to 0–100. Bot measurements: answering after 0.3 s on every question ≈ 95–97, after 4.5 s ≈ 68,
  all wrong / no input = 0. The whole run takes about `sec` seconds even with no input.
- Mini mode writes nothing: `saveHistory()` at startup is skipped, `save()`/`answer()` are never called, no `PlayCounts`.
  There is no sound in this game, so `mute=` is ignored.
- The title's `art/keyart.png` (2.4 MB) and `art/intro.mp4` (1 MB) are not needed in mini mode; their `src` is removed on load,
  but the browser may already have started fetching them.
- `window.__quiz` (mini mode only) exposes `correct()` / `locked()` for automated tests. Standalone check:
  `?mini=1&sec=10` auto-starts after 300 ms (`window.parent === window`).
