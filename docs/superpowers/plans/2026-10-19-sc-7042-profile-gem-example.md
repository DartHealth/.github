# SC-7042 Profile Gem Example Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the `data_nexus` usage example on github.com/DartHealth run when copied into a plain Ruby script, and stop with a clear message when no program matches.

**Architecture:** A two-line documentation change to `profile/README.md` in the `DartHealth/.github` repo: load the gem at the top of the example, and restore the upstream "Program not found" guard after the program lookup. Verification is a throwaway harness outside the repo that extracts the example from the README and runs it against the `data_nexus` v0.3.0 source with HTTP stubbed. Nothing in the harness is committed.

**Tech Stack:** Markdown, Ruby 3.4 (local), `data_nexus` gem v0.3.0 source, `gh` CLI.

**Spec:** Shortcut [SC-7042](https://app.shortcut.com/dart/story/7042). Its first comment, "Investigation notes", has the citations.

## Global Constraints

- Repo: `~/src/darthealth/.github` (remote `git@github.com:DartHealth/.github.git`). Branch from `origin/main` (at `4325949` or later).
- Branch name: `briansage/sc-7042/profile-gem-example` (the `sc-7042` links it to the card).
- Brian's global git config sets `push.default tracking`. Branch with `--no-track` and push with the full `HEAD:refs/heads/<branch>` refspec. A branch that tracks `origin/main` pushes to `main`.
- Change only the Ruby usage example under `## Ruby gem: \`data_nexus\``. No other wording, links, or sections change.
- The no-match guard text is exactly `raise "Program not found: Example Program" unless program`. That's the line in the data_nexus 0.3.0 README and CHANGELOG.
- Never call the real Data Nexus API. Every run uses the stub harness and the fake key `test-key`.
- Commit messages end with `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`. PR descriptions end with `🤖 Generated with [Claude Code](https://claude.com/claude-code)`.
- Serial (Oxford) commas in all prose: the PR description and the card comment.
- `~/.gitignore` ignores dotfiles. This change touches no dot-paths, so `git add profile/README.md` is enough.

## Review Focus

- **Rails, or any app that already loaded the gem through `Bundler.require`:** the added `require` must be a harmless no-op. Task 1, Step 6 runs the example with the gem preloaded.
- **A key that can see the program, but the name differs in case or surrounding whitespace:** the API ignores both, so the example still finds the program. That's API behaviour, not README behaviour, and it can't be tested offline. It's out of scope; don't add handling for it.
- **`DATANEXUS_API_KEY` unset:** `DataNexus::Client.new` raises `DataNexus::ConfigurationError`, which is a clear message. Nothing to change.
- **GitHub rendering:** the fenced block has to stay a single ```ruby block, or the extraction in Step 2 will fail. Task 2, Step 4 checks the rendered page after merge.

---

### Task 1: Fix the profile README example

**Files:**
- Modify: `profile/README.md`, the second ```ruby block under `## Ruby gem: \`data_nexus\`` (the block that starts `client = DataNexus::Client.new(...)`)
- Test (outside the repo, not committed): `$H/stub_http.rb` and `$H/extract_block.rb`, where `H="${TMPDIR:-/tmp}/readme-check-sc-7042"`

**Interfaces:**
- Consumes: the `data_nexus` v0.3.0 source from `~/src/darthealth/datanexus-ruby` (tag `v0.3.0`)
- Produces: the commit on branch `briansage/sc-7042/profile-gem-example` that Task 2 pushes

- [ ] **Step 1: Branch and build the harness**

```bash
cd ~/src/darthealth/.github
git fetch origin
git switch --no-track -c briansage/sc-7042/profile-gem-example origin/main

H="${TMPDIR:-/tmp}/readme-check-sc-7042"
mkdir -p "$H"
git -C ~/src/darthealth/datanexus-ruby worktree add --detach "$H/gem" v0.3.0
```

Write `$H/stub_http.rb`:

```ruby
# Loaded with `ruby -r`. Defines nothing in DataNexus, so a script that never
# requires the gem still hits NameError. Once the script requires 'data_nexus',
# DataNexus::Connection#get/#post are replaced so no request leaves the machine.
# STUB_PROGRAMS=match (default) returns one program; STUB_PROGRAMS=none returns [].
module DataNexusHttpStub
  def get(_path, _params = {})
    programs = ENV.fetch('STUB_PROGRAMS', 'match') == 'match' ? [{ id: 'p-1', name: 'Example Program' }] : []
    { data: programs, has_next_page: false, has_previous_page: false }
  end

  def post(_path, _body = {})
    { data: [{ first_name: 'Jane', last_name: 'Doe' }], more_results: false }
  end
end

module StubOnRequire
  def require(name)
    super.tap do
      if name == 'data_nexus' && !(DataNexus::Connection < DataNexusHttpStub)
        DataNexus::Connection.prepend(DataNexusHttpStub)
      end
    end
  end
end
Kernel.prepend(StubOnRequire)
```

Write `$H/extract_block.rb`:

```ruby
# Usage: ruby extract_block.rb README.md "## Heading" N
# Prints the Nth (1-based) ```ruby block that follows the heading line.
path, heading, n = ARGV
lines = File.readlines(path)
start = lines.index { |l| l.chomp == heading } or abort("heading not found: #{heading}")
blocks = []
cur = nil
lines[(start + 1)..].each do |l|
  if cur.nil? && l.start_with?('```ruby') then cur = +''
  elsif cur && l.start_with?('```') then blocks << cur; cur = nil
  elsif cur then cur << l
  end
end
block = blocks[Integer(n) - 1] or abort("block #{n} not found")
print block
```

- [ ] **Step 2: Run the example to verify it fails**

```bash
H="${TMPDIR:-/tmp}/readme-check-sc-7042"; cd ~/src/darthealth/.github
ruby "$H/extract_block.rb" profile/README.md '## Ruby gem: `data_nexus`' 2 > "$H/example.rb"
out=$(DATANEXUS_API_KEY=test-key ruby -I "$H/gem/lib" -r "$H/stub_http.rb" "$H/example.rb" 2>&1); echo "exit=$?"; printf "%s\n" "$out" | head -1
```

Expected: `exit=1`, then `example.rb:1:in '<main>': uninitialized constant DataNexus (NameError)`

- [ ] **Step 3: Write the minimal fix**

In `profile/README.md`, change the block from:

```ruby
client = DataNexus::Client.new(api_key: ENV['DATANEXUS_API_KEY'])

# Find a program by name, then check whether someone is an eligible member
program = client.programs.list(name: 'Example Program').data.first

result = client.programs(program[:id]).search_members(
```

to:

```ruby
require 'data_nexus'

client = DataNexus::Client.new(api_key: ENV['DATANEXUS_API_KEY'])

# Find a program by name, then check whether someone is an eligible member
program = client.programs.list(name: 'Example Program').data.first
raise "Program not found: Example Program" unless program

result = client.programs(program[:id]).search_members(
```

Leave the rest of the block (the `search_members` arguments and the `result[:data].each` line) unchanged.

- [ ] **Step 4: Run the example to verify both cases pass**

```bash
H="${TMPDIR:-/tmp}/readme-check-sc-7042"; cd ~/src/darthealth/.github
ruby "$H/extract_block.rb" profile/README.md '## Ruby gem: `data_nexus`' 2 > "$H/example.rb"
DATANEXUS_API_KEY=test-key ruby -I "$H/gem/lib" -r "$H/stub_http.rb" "$H/example.rb"
out=$(DATANEXUS_API_KEY=test-key STUB_PROGRAMS=none ruby -I "$H/gem/lib" -r "$H/stub_http.rb" "$H/example.rb" 2>&1); echo "exit=$?"; printf "%s\n" "$out" | head -1
```

Expected, the first command: `Jane Doe`
Expected, the second command: `exit=1`, then `example.rb:7:in '<main>': Program not found: Example Program (RuntimeError)`

- [ ] **Step 5: Confirm the diff is exactly two added lines plus one blank line**

```bash
cd ~/src/darthealth/.github && git diff --stat && git diff
```

Expected: `profile/README.md | 3 +++`, with `require 'data_nexus'`, a blank line, and the `raise` line added. Nothing removed.

- [ ] **Step 6: Check the Review Focus case for an app that already loaded the gem**

```bash
H="${TMPDIR:-/tmp}/readme-check-sc-7042"
DATANEXUS_API_KEY=test-key ruby -I "$H/gem/lib" -r "$H/stub_http.rb" -e "require 'data_nexus'; load '$H/example.rb'"
```

Expected: `Jane Doe` (the second `require` returns `false` and does nothing)

- [ ] **Step 7: Commit**

```bash
cd ~/src/darthealth/.github
git add profile/README.md
git commit -m "$(cat <<'EOF'
Load the gem and guard the program lookup in the profile's Ruby example

Copied into a plain script, the example raised NameError because it never
required data_nexus, and NoMethodError when no program matched the name.
The no-match guard matches the data_nexus 0.3.0 README (sc-7042).

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
EOF
)"
```

### Task 2: Open the PR, post the card status, and clean up

**Files:** none in the repo.

**Interfaces:**
- Consumes: the commit from Task 1
- Produces: the PR URL, which is posted on SC-7042

- [ ] **Step 1: Push and open the PR**

```bash
cd ~/src/darthealth/.github
git push -u origin HEAD:refs/heads/briansage/sc-7042/profile-gem-example
gh pr create --base main --title "Load the gem and guard the program lookup in the profile's Ruby example (sc-7042)" --body "$(cat <<'EOF'
## Summary

The `data_nexus` example on github.com/DartHealth failed when copied into a plain Ruby script:

- It never required the gem, so it raised `NameError: uninitialized constant DataNexus`.
- When no program matched the name, `program[:id]` raised `NoMethodError` on nil.

This adds `require 'data_nexus'` and restores the `raise "Program not found: Example Program" unless program` guard from the data_nexus 0.3.0 README.

## Verification

Extracted the example from the README and ran it against the data_nexus v0.3.0 source, with HTTP stubbed and no real API calls:

- With a matching program, it prints the stubbed member.
- With no matching program, it raises `Program not found: Example Program`.
- With the gem already loaded (as in Rails), the extra `require` does nothing.

Shortcut: [sc-7042](https://app.shortcut.com/dart/story/7042)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

Expected: `gh` prints the PR URL.

- [ ] **Step 2: Post the PR-opened status comment on SC-7042**

Engage `dart-shortcut:shortcut-cards` first. Using the `shortcut-dart` MCP server, add a comment on story 7042 with exactly this text, filling in the URL from Step 1:

```
PR opened: <PR URL from Step 1>

_Posted by Claude for Brian Sage_
```

Don't change the card's state. Moving it is Brian's call.

- [ ] **Step 3: Remove the harness worktree**

```bash
H="${TMPDIR:-/tmp}/readme-check-sc-7042"
git -C ~/src/darthealth/datanexus-ruby worktree remove "$H/gem"
rm -rf "$H"
git -C ~/src/darthealth/datanexus-ruby worktree list
```

Expected: the list shows only the main checkout.

- [ ] **Step 4: After merge, check the rendered page**

Open https://github.com/DartHealth and confirm the Ruby example shows `require 'data_nexus'` on its first line and the `raise` line under the lookup, inside one code block.
