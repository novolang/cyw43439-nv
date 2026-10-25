# Changelog

## 0.1.1 — 2026-10-08

The firmware blobs travel with the package: 0.1.0's archive carried
none, since the publish packed only the directories it knew by name,
and a build of it refused the `[assets]` table's first path.  The
toolchain now packs every path the table names.  0.1.0 is withdrawn.

