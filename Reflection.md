# Reflection
 
## 1. What did a fresh session get wrong or ask about that an earlier round had already settled? What was missing from the repository that would have prevented it?
 
In round 3 of the dungeon crawler the fresh session said that "problem.md has no Round 3 requirements" when the round 3 requirements were already committed (commit `b3ad86f`). This happened because I forgot to pull the changes into my local copy. This could've been prevented if there were a line in `CLAUDE.md` that told it to run `git pull` before starting.
 
## 2. In the round that changed an earlier requirement, what broke? Did the tests catch it, and does the decision record explain the change to someone who was not there?
 
In round 3 I changed R3 so that new levels could be added and the player only won after making it to the exit of the last level. The session kept `Game.from_map` as a one-level game, so the old R3 tests passed without testing the new rule. The new rule was implemented in commit `8c8d744`, but the tests were still using the old rules. In `2b5c116` the old R3 tests were rewritten to `test_r3_reaching_exit_wins_only_on_last_level`, and the old rule, when put back, caused 4 tests to fail.
 
Decision record `docs/decisions/0006-win-on-last-level-exit.md` explains the change pretty well to someone who was not there, because the Background section clearly states what the previous rounds did and what round 3 added, and the Decision section gave context for all of Claude Code's changes. For example, it gave context for making the exit of the last level the only win, stating that it allowed the earlier levels' exits to load the next level and keep the game running.
 
## 3. Name one test that caught a real problem, or one test the assistant wrote that checked nothing useful. Quote it.
 
A test that checked nothing useful was `test_r3_reaching_exit_wins` in `tests/test_r3_win_loss.py`, written before round 3 and later replaced in `2b5c116`:
 
```python
def test_r3_reaching_exit_wins():
    """R3.1: stepping onto the exit sets status 'won'."""
    game = Game.from_map(["@.X"])
    game.move("right")
    assert game.status == "playing"
    message = game.move("right")
    assert game.status == "won"
    assert game.player == (0, 2)
    assert "win" in message.lower()
```
 
As a result, when the rule changed the test kept passing, because `Game.from_map(["@.X"])` only built one level, making the first exit the last.
 
## 4. Choose one commit you would now write differently, and say how.
 
I would write the commit `87b46be` "Update problem.md" differently, because all I did was use the GitHub default message. I should've been more specific about what I updated. I'd change it to "Updated the problem section to be more specific to what the problem is."
 
## 5. Choose one function in the project you developed further that you did not read while building it. Without an assistant open, read it and write what you think it does. Then ask an assistant, and say what you got right and wrong.
 
`_paralysis_tick` in `dungeon/game.py` (lines 280–289, docstring left out here):
 
```python
def _paralysis_tick(self) -> tuple[bool, str]:
    if self.paralysis == 0:
        return False, ""
    self.paralysis -= 1
    return True, " The paralysis wears off." if self.paralysis == 0 else ""
```
 
**My analysis (written before asking an assistant):** if the paralysis counter is 0 return false and than subtract the counter by one. Once thats done return true to turn off the paralysis
 
**What I got right:** Returning false if the counter is zero.
 
**What I got wrong:**
 
- The subtraction only happens when the counter is not 0. The return stops the function first.
- `True` means "you are paralyzed this turn", not "turn it off". Paralysis "turns off" when the counter reaches 0, and the function then adds the "wears off" message.
 
