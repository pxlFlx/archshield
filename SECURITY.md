# Security policy

Please report suspected security issues privately to the repository maintainers rather than opening a public issue with an exploit. Include the affected version, impact, and a minimal reproduction that does not contain private repository data.

ArchShield treats scanned repositories as untrusted input. It parses source without executing it, does not install target-project dependencies, does not follow symlinks by default, enforces a file-size limit, and does not transmit source code. Keep ArchShield and its dependencies updated.
