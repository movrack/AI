# Security policy

If you find a vulnerability (for example a script that writes outside its target folders, deletes files, or leaks
data), do **not** open a public issue. Use GitHub's private vulnerability reporting: **Security → Report a
vulnerability** on this repository. You will get an answer from the owner as soon as possible.

The agents and the `install-agents` script never delete files, never write outside the assistant folders they are
asked to update, and back up any file they did not generate before replacing it.
