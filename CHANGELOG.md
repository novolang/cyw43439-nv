# Changelog

## 0.1.3 — 2026-11-03

This release requires novo 0.23.0 or later.

## 0.1.2 — 2026-10-25

A call that passes a variable to a `var` parameter now writes the `var` marker, which the 0.21.1 compiler asks for (E2050).  The package needs Novo 0.21.1 or later.  No signature and no behaviour changed.

## 0.1.1 — 2026-10-08

The firmware blobs travel with the package: 0.1.0's archive carried
none, since the publish packed only the directories it knew by name,
and a build of it refused the `[assets]` table's first path.  The
toolchain now packs every path the table names.  0.1.0 is withdrawn.
