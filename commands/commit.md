It seems we are ready to commit. Before doing so, carefully follow these steps in order: 1. Code cleanup with RuboCop
• Run RuboCop on the entire codebase.
• Fix every single violation, no matter how small or unimportant it may seem.
• Do not add # rubocop:disable or any similar directive — always fix the underlying problem properly. 2. Run the full test suite
• Execute rspec spec.
• This may take a long time — keep running until you get a completely clean run with all tests passing.
• If any tests fail, fix the code accordingly and re-run until there are zero failures. 3. Commit preparation
• Once RuboCop and RSpec are both clean, prepare the commit.
• Write a concise commit message that summarizes the changes clearly.
• The message must not mention Claude, AI, or any tooling.
• Ensure the commit is authored under the correct committer name and email — do not include Claude as a committer.

Only after all of this is complete should you actually perform the commit.
